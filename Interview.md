# Interview

Note for interview

---

## Linux

### most useful commands
- cd, cp, vim, top, ps, grep, awk, sed, tr, free -h
- grep -A5 -B5 "error" /var/log/syslog => get 5 line after and 5 line before of match
- `awk -F ":" '{ print $1 }'` /etc/passwd
- tr -d ':' => only std input, translate
- sed => -i in-place, -E,r extended re, -n, -i.back
    - sed 's/root/navaneeth/g' # s substitute, g global
    - sed '3d' text.txt
    - sed -n '3,5p' test.txt

### Troubleshooting
- USE => utlilization, saturation, error
- logs, journelctl -u <unit> --since "1 hour ago" --no-pager or tail /var/log/syslog
- top, M => mem, P => cpu
- ps aux, ps -e -o pid,%cpu,%mem,cmd --sort %cpu
- get more info from /proc/<pid>
- Identify the issue 

### CPU
- always time
- %wa => wait IO average => ^| issue with storage
- load avg, avg number process is running or waiting in cpu, 
1m 5m 15m, useful with number of cores

### Mem
- free -h
- OOM kills
  - got error and when checked free -h all looks normal =? 1. process limit, 2. during the time the error got generated mem usage was at peak and later it came down, check dashboard and confirm
- most of the case culprit is app, poorly managed memory intensive activities

### Process
- ps or top
- type of process state
    - R -> running, or sitting in queue
    - S -> Interpretable sleeping, paused waiting for something, nw stream or user input
    - D -> Unintereptable sleep, waiting for hardware operation (like disk read), it ignore SINGALs and canot be killed
    - T -> stopped, paused using SIGSTOP or SIGSTP, resume with SIGCONT
    - Z -> Zombie, process finished and exited, but remains in the process table it wait for parent read its final status
    can be killed, parent assign to init, init calls .wait() and remove from process table
- troubleshoot
    - D => disk or storage IO mostly

### SIGNAL
- method to interrupt a process
- SIGTERM = signal to terminate gracefully, catchable and ignorable, kill, 
- SIGKILL = kill immediately, non catchable and ignorable => 9
- SIGSTOP = to change to T state, paused, non catchable and ignorable
- SIGCONT = resume form T state
- SIGINT = interrupt, ctrl + c
- SIGSTP = same like SIGSTOP, but initiate when ctrl + z, non catchable and ignorable
- SIGHUP = terminate and reload

### Kernel

### selinux

### file perm
- `x` perm in dir is not execute, it is the access for go inside the dir
- 4 => read, 2 => write, 1 => execute
- special bit
    setuid => run as user => s for user => 4
    setgid => run as group => s for group => 2
    sticky => if enabled only file owner can delete => t => 1

### lsof - List of open files
- everything is file in linux
- lsof +D /mnt/usb
- lsof | grep deleted # lsof +L1
- lsof -p <pid>


### networking
- ss -tulpn or nestat -tulpn
- check firewall status always
    - service firewalld
    - firewall-cmd --state
    - firewall-cmd --zone=public --add-port=5000/tcp --permanent && firewall-cmd --reload # or --add-service
- iptable -> old
- route, ip add route <dest>/<cidr> via <gateway> dev <interface>

### Links
- Hard => create a link file that point to the inode of the an existing file.
    - even if we delete original file it will remain in the disk
    - inode will be deleted only after we delete all hard links
    - dir not supported
- Soft => symbolic link, it will create a file that containing path to an existing file
---

## docker
Docker is a platform that is used to build, manage, run containerized applications.
Containerized app meaning, it is an isolated unit that has application code + its dependencies packed together and called as a container image
Docker is a linux process run in an isolated kernel namespace

**Adv**
- isolated
- solves it works in my env issue
- it work similar everywhere

### architecture
- Docker client, `docker run` it communicate to docker host/server through api
- Docker host
    - dockerd -> deamon manage user auth, policies, nw routing
    - containerd -> manage lifecycle of containers
    - runc -> actual OCI runtime
    - Linux kernel => namespace for isolation, cgroup for resource limit, nproc
- namespace is kernel feature that give a process an isolated env and it create an illution it has their own system

### basic commands
- docker run -d -v <localpath>:<container_path> -p <localport>:<containerPort> --name web-01 nginx:<tag> <optional_arg_or_cmd>
- docker build -t <name>:<tag> .
- docker pull
- docker rm <container>
- docker rmi <image>

### troubleshoot
- docker ps
- docker logs <container>
- docker stats

### docker vs compose
- docker 
    - allow us to run a single container using run command
    - it will be attached to the default network docker0
    - they can communicate each other with ip, not with name
- compose 
    - is yml manifest that groups related containers as service
    - it is create its own network and services can communicate each other with ip and service name

### networking
- bridge
- host

### volume
- named volumes
- Bind mount, user mount
- RAM mem, Tmpfs

---

## k8s
Container orchestration platform

### Architecture
1. Control plane - Brain
    - API server, share state
    - etcd => store desired state, database like, key-value store
    - scheduler => it watches API server, if new pod has no node assigned it find a best node and write back to API
    
    - controller => reconcile, keep watching if a pod dies, it will create a new resource match the desired state in etcd
2. Worker nodes - muscle
    - kubelet => listen API server and start the container
    - kubeproxy => handle networking rules to reach pods

### Load balancer
- cloud load balancer
- MetalLB => for private, create with IP of internal network eg: 192.168.1.4 for each service
- external traffic

### Ingress
- traffic director
- resources => decalare traffic routing rules
- controller => process to rules and route the traffic

### service
- pods are temporary, they crash and come up new, scale, move to nodes, gets new ip/s
- solve this problem, service give permenent IP and DNS name, and this route to correct pod using pod selector
- automatic load balancing
- Type of service:
    - Cluster IP => only with in the cluser
    - NodePort => acessible via node port
    - LoadBalancer => access externally

### Troubleshoot
- Pod status is pending
    - it has accepted by the scheduler but hasnt placed a node
    - it could be due to resource issue, PVC, affinity, ConfigMap or secret missmatch
    - kubectl describe pod <name>
- CrashLoopBack
    - Pod is assigned to a node
    - Imagepull error
    - app starting issue, dependency
- Completed
    - container process is exiting with code 0
    - need to check the commands running using describe

### Access
- n/w, kubeconfig -> tells kubectl, where the cluster is and who is connecting
- Auth -> API server checks indentity throgh RBAC
- 2 ways
  - ServiceAccount -> user stays in cluster, token based auth -> SA -> Role -> RoleBinding
    - generate token -> kubectl create token <sa> --duration=72h 
    - create kubeconfig.yml using token, CA, and cluster info
    - share to user
  - Certificate based user -> user has a signed certificate by k8s CA, CN=username, O=group
    
### commands
- kubectl logs <pod> --previous
- kubectl exec -it <pod> -- <command>   # curl if check the status locally
- kubectl rollout restart deployment <name>

### rollout
- RollingUPdate => zero downtime
- Recreate => remove and create new, downtime

### deployment vs stateful set
- Deployment
    - pod names are dynamic and some random hash value will be added
    - deployment is parellel
    - storage ephemeral storage
- Statefuset
    - strict naming, db-01, db-02
    - Deployment is oredered
    - each pod bounds to pvc storage, if recreated it will bound same

---

## Golang

### Concurrency
- dont communicate by sharing mem, share mem by communicating
- go achieve concurrency through go routine and sync with channel
- go routine is a function that run councurrently with other function
- go func(){}

### wait groups, sync
- problem: in some scenario the main program exit earlier than before go routine finishes
- wait group fix this by waiting go routine to complete
- var wg sync.WaitGroup => wg.Add(1) => wg.Done() (inside goroutine) => wg.Wait() (in main or)

### channel
- through which goroutines securly send and receive data
- ch := make(chan string) => pass to goroutine => ch <- "done" => state := <-ch 

### select
- it blocks until one of its channel case is ready to execute
```go
select {
case msg1 := <-channelA:
    fmt.Println("Received from A:", msg1)
case msg2 := <-channelB:
    fmt.Println("Received from B:", msg2)
case <-time.After(time.Second * 2):
    fmt.Println("Network request timed out!") // Fallback safety pattern
}
```

### Interface
- is a type that define set of method structure, blueprint or contact
- any type with those metrics satisfies the interface
- interface{}/any

### struct, methods, struct tag
- type User struct{ id int }
- func (u User) Save() {}
- tag, `json:"id"`

### json handling
- encoding/json - module
- json.Marshal()
- json.UnMarshal()

### file handling
- os.WriteFile()
- os.ReadFile()

### error handling
- unlike other programming lang go handle error as vaule

### datatypes
- int, unit, float32, float64
- string
- byte
- bool
- array [2]string
- slice []string
- map[string][string], make or literal

### type conversion
- string() -> to string
- []byte() -> to byte array
- strconv -> str to other type, "0.3" to float

### logging
- log package
- modern log/slog => slog.Info()

### gin
1. router := gin.Default()
2. router.GET("/ping", func(c *gin.Context){
    c.JSON(http.StatusOK, gin.H{"msg": "pong"})
})
3. router.Run(":8080")

---

## Python

### tricky questions
- is vs == -> check the idendity vs check equality
- None vs "" -> no value assigned vs value assigned but empty string
- func args initialize during creation, it will update each call, cuacious with collection types
- list_b = list_a, share same mem, if one update another also, if you want a sep copy use .copy() or deepcopy (for nested elements)
- dict .get() vs my_dict[], safe return null if not there vs error if key not found 

### package management

### Decorator
- as name indicate it decorate a function, ie modify its behaviour without changing the code

### Generator
- generator is a func to create itratable object
- it use a special kw yield, executed using next(), it pause the execution for each iteration and remember its state

### context
- it properly manage the closure of an object
- with, example open file

### FastAPI
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(version="1.0")

class User(BaseModel):
    id: int
    name: string
    
users: list[User] = []

@app.get("/user/{id}")
def get_user(id: int):
    for u in users:
        if u.id == id:
            return u
            
    raise HTTPException(status_code=404, detail="user not found")
    

@app.post("/user")
def create_user(u: User):
    users.append(u)
    return {"msg": "user created"}

if __name__ == "__main__":
    app.run(":8080")
    
```

### Requests

### List comprehension

```python
[ x * x for x in my_list ]
```

### pandas

```python
import pands as pd

data = {
"ids": [1, 2, 4],
"name": ["a", "b", "c"]
}

pd.DataFrame(data)
```

### std libraries, path, datetime, json, logging, file operation

```python
import datetime
import logging
from os import path

logging.baseConfig(loglevel=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("app started")

cwd = path.dirname(__file__)
conf = path.join(cwd, "config.ini")

now = datetime.datetime.now()
after_2days = now + datetime.timedelta(days=2)

```

### error handling

### boto3

---

## Ansible

### ad-hoc commands

### palybook

### role

### module
- std
- custom

### ansible conf

### inventory file

### GUI

---

## Prometheus
Prometheus is monitoring and alerting tool and best suit for cloud native environment
pull model
metrics stored as a timeseries data

### scrap configuration
```yml
global:
    scrape_interval: 15s

scrape_config:
    - job_name: node_exporter
      scrape_timeout: 10s
      static_configs:
        - targets: ["192.168.1.50:9000"]
          labels:
            env: prod
    
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
        - role: pod

      relabel_configs:
        - source_labels:
            [__meta_kubernetes_namespace]
          action: keep
          regex: monitoring

```
### k8s
- service discovery, kubernetes_sd_config
- role => pod, node, endpoints, service
- relabel is to filter, replacement (ip with dns)
- kube-state metrics, prom community maintained service
- kubelet metrics
- cAdvisor metrics, container level

### histogram
- measure the distribution of value instead of storing every value, eg: req latency
- prometheus observe and store in buckets, buckets are cumulative
- it expose
    - <metric_name>_bucket{le=""}
    - <metric_name>_bucket{le=""}
    - <metric_name>_bucket{le=""}
    - <metric_name>_bucket{le="+Inf"}
    - <metric_name>_sum
    - <metric_name>_count
- 99th percentile, eg 99% of request complete with in X second
- avg - single number dosent represent the most of the people experience, meaning is hidden

### some query example
- rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
- histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
- (
  sum by (container) (rate(container_cpu_usage_seconds_total{container="my-app-container"}[5m]))
  /
  sum by (container) (container_spec_cpu_quota{container="my-app-container"} / container_spec_cpu_period{container="my-app-container"})
) * 100
- container_memory_used_bytes

### cAdvisor
- as a systemd service

### docker monitoring

### node exporter

## SRE
if you are not measuring you are not operating

## terminologies
- SLO
- automate toil
- blast radius
- 

## Suggestion for builds
- should be reproduceble, by pinning dependency using dock files, --no-cahe, should build in a seperate build env rather than dev laptop etc
- treat ci-cd as equivalant to production deployment and track through SLO
- parallel CI jobs
- use cache where ever possible
- build once, promote to dev -> stagging -> prod
- produce rollback ready artifacts

## Improve build speed
- run tasks parellel
- use cache
- pre baked runner images
- pre baked builer image
- if it is new service try to use go lang, it is created for speed
