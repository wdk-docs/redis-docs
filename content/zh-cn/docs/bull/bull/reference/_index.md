---
title: "参考"
linkTitle: ""
weight: 4
---

- [Queue](#queue)

  - [Queue#process](#queueprocess)
  - [Queue#add](#queueadd)
  - [Queue#addBulk](#queueaddBulk)
  - [Queue#pause](#queuepause)
  - [Queue#isPaused](#queueispaused)
  - [Queue#resume](#queueresume)
  - [Queue#whenCurrentJobsFinished](#queuewhencurrentjobsfinished)
  - [Queue#count](#queuecount)
  - [Queue#removeJobs](#queueremovejobs)
  - [Queue#empty](#queueempty)
  - [Queue#clean](#queueclean)
  - [Queue#obliterate](#queueobliterate)
  - [Queue#close](#queueclose)
  - [Queue#getJob](#queuegetjob)
  - [Queue#getJobs](#queuegetjobs)
  - [Queue#getJobLogs](#queuegetjoblogs)
  - [Queue#getRepeatableJobs](#queuegetrepeatablejobs)
  - [Queue#removeRepeatable](#queueremoverepeatable)
  - [Queue#removeRepeatableByKey](#queueremoverepeatablebykey)
  - [Queue#getJobCounts](#queuegetjobcounts)
  - [Queue#getCompletedCount](#queuegetcompletedcount)
  - [Queue#getFailedCount](#queuegetfailedcount)
  - [Queue#getDelayedCount](#queuegetdelayedcount)
  - [Queue#getActiveCount](#queuegetactivecount)
  - [Queue#getWaitingCount](#queuegetwaitingcount)
  - [Queue#getPausedCount](#queuegetpausedcount)
  - [Queue#getWaiting](#queuegetwaiting)
  - [Queue#getActive](#queuegetactive)
  - [Queue#getDelayed](#queuegetdelayed)
  - [Queue#getCompleted](#queuegetcompleted)
  - [Queue#getFailed](#queuegetfailed)
  - [Queue#getWorkers](#queuegetworkers)

- [Job](#job)

  - [Job#progress](#jobprogress)
  - [Job#log](#joblog)
  - [Job#getState](#jobgetstate)
  - [Job#update](#jobupdate)
  - [Job#remove](#jobremove)
  - [Job#retry](#jobretry)
  - [Job#discard](#jobdiscard)
  - [Job#promote](#jobpromote)
  - [Job#finished](#jobfinished)
  - [Job#moveToCompleted](#jobMoveToCompleted)
  - [Job#moveToFailed](#jobMoveToFailed)

- [Events](#events)
  - [Global events](#global-events)
