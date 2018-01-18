---
title: "harbor 源码分析： job"
date: 2017-12-19T18:45:17+08:00
tags: ["docker", "harbor", "job"]
categories: 云计算
slug: harbor-job.md
type: post
toc: true
---


## 概述
harbor 中有两个常用的功能，`镜像复制`和`镜像扫描`，而这两个操作都是比较耗时的任务，在 harbor 中，被设计为 job 来完成。


## 接口定义
同样的，我们先来看 Job 的接口定义

代码在 `github.com/vmware/harbor/src/jobservice/job/jobs.go`

```go
type Job interface {
    // job 的 id，通过 id 可以查看对应的信息
    ID() int64
    // job 类型，目前只有 Scan 和 Replication 两种
    Type() Type
    // 日志所在路径，通过这个路径可以查看 job 的状态
    LogPath() string
    // 更新 job 状态
    UpdateStatus(status string) error
    // 初始化一个 job  
    Init() error
}
```


## 镜像扫描

### 1. 镜像扫描任务的基本结构

代码在 `github.com/vmware/harbor/src/jobservice/job/jobs.go`

```go
type ScanJob struct {
	id   int64
	parm *ScanJobParm
}

type ScanJobParm struct {
	Repository string
    Tag        string
    // 镜像 哈希值
	Digest     string
}
```

镜像扫描需要的参数很少，其实就是镜像的地址，附带一个镜像哈希值。

### 2. 任务的产生
镜像扫描任务的来源有两个，一个是定时扫描任务，一个是手工点击的扫描任务。

代码在 `github.com/vmware/harbor/src/jobservice/api/scan.go`

```go
func (isj *ImageScanJob) Post() {
	var data models.ImageScanReq
	isj.DecodeJSONReq(&data)
	log.Debugf("data: %+v", data)
	repoClient, err := utils.NewRepositoryClientForJobservice(data.Repo)
	if err != nil {
		log.Errorf("An error occurred while creating repository client: %v", err)
		isj.RenderError(http.StatusInternalServerError, "Failed to repository client")
		return
	}
	// 通过镜像地址获取镜像的哈希值
	digest, exist, err := repoClient.ManifestExist(data.Tag)
	if err != nil {
		log.Errorf("Failed to get manifest, error: %v", err)
		isj.RenderError(http.StatusInternalServerError, "Failed to get manifest")
		return
	}
	if !exist {
		log.Errorf("The repository based on request: %+v does not exist", data)
		isj.RenderError(http.StatusNotFound, "")
		return
	}
	// 在数据库中创建一条记录，记录某次扫描的 镜像信息
	j := models.ScanJob{
		Repository: data.Repo,
		Tag:        data.Tag,
		Digest:     digest,
	}
	jid, err := dao.AddScanJob(j)
	if err != nil {
		log.Errorf("Failed to add scan job to DB, error: %v", err)
		isj.RenderError(http.StatusInternalServerError, "Failed to insert scan job data.")
		return
	}
	log.Debugf("Scan job id: %d", jid)
	// 创建一个 扫描的任务
	sj := job.NewScanJob(jid)
	log.Debugf("Sent job to scheduler, job: %v", sj)
	// 把扫描任务放到任务队列中
	job.Schedule(sj)
}
```

### 3. 任务的消费者是谁

代码在 `github.com/vmware/harbor/src/jobservice/job/scheduler.go` 中

```go
var jobQueue = make(chan Job)

func Schedule(j Job) {
	jobQueue <- j
}
```

我们看到了 Schedule 的实现，其实就是把 job 放到了 job queue 中，
那么 jobQueue 是有谁来处理的呢？ 我们可以直接搜索 jobQueue

在 `github.com/vmware/harbor/src/jobservice/job/workerpool.go` 中

```go
func Dispatch() {
	for {
		select {
		case job := <-jobQueue:
			go func(job Job) {
				log.Debugf("Trying to dispatch job: %v", job)
				worker := <-WorkerPools[job.Type()].workerChan
				worker.Jobs <- job
			}(job)
		}
	}
}
```

这里我们特别注意一下 worker，首先 worker 是有 type 的，而且 type 和 job 对应，我们之前就说过，当前 job 的 type 有 `Scan` 和 `Replication` 两种。

在 `github.com/vmware/harbor/src/jobservice/job/workerpool.go` 中

```go
type Worker struct {
	ID    int
	Type  Type
	Jobs  chan Job
	queue chan *Worker
	SM    *SM
	quit  chan bool
}

// Start is a loop worker gets id from its channel and handle it.
func (w *Worker) Start() {
	go func() {
		for {
			w.queue <- w
			select {
			case job := <-w.Jobs:
				log.Debugf("worker: %v, will handle job: %v", w, job)
				w.handle(job)
			case q := <-w.quit:
				if q {
					log.Debugf("worker: %v, will stop.", w)
					return
				}
			}
		}
	}()
}
```

当接收到 job 的时候，会交给 worker 的 handle 函数处理，具体的处理我们暂时不分析。
到此，一个简单的生产者、消费者模型完成了。


## 总结
上面分析镜像扫描任务的部分代码，同理可得，镜像复制其实也是一样的处理，只是 任务的参数，任务的处理方式不同罢了。  
如果需要更深入的分析，那么先要了解 `状态机` 这个概念，这里我就不继续了。有兴趣的同学可以自己尝试分析，如果有什么问题也可以和我一起交流讨论。
