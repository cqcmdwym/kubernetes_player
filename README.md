# This is the guideline tell you how to learn `Kubernetes`

## Environment setup on MacOS
https://deis.com/blog/2016/run-kubernetes-on-a-mac-with-kube-solo/
https://deis.com/blog/2015/zero-to-kubernetes-dev-environment-on-os-x/
1. Don't forget to install Corectl APP
2. libev `brew install libev`
3. **Up** the KubeSolo menu
4. **Updates/Update macOS helm and deis clients** to force install helm and deis(Sometimes those tools not installed by **up** command)
ref:https://github.com/mist64/xhyve
ref:http://www.iterm2.com/#/section/downloads

## Enviroment setup on VirtualBox
1. Install VirtualBox
2. Download CentOS7 image and [setup on VirtualBox](http://resources.infosecinstitute.com/installing-configuring-centos-7-virtualbox/http://resources.infosecinstitute.com/installing-configuring-centos-7-virtualbox/)
3. Turn Firewall
```sh
systemctl disable firewalld
systemctl stop firewalld
```
4. Install **etcd** and *kubernetes*(will install Docker together)
```sh
yum install epel-release
yum install -y etcd kubernetes
```
5. 安装完毕后，需要修改如下两个配置文件： 
* 修改/etc/sysconfig/docker，将其中OPTIONS内容设置为：OPTIONS=’–selinux-enabled=false –insecure-registry grc.io’； 
* 修改/etc/kubernetes/apiserver，把–admission-control参数的ServiceAccount删除
6. Start services
```sh
systemctl start etcd
systemctl start docker
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kuberlet
systemctl start kube-proxy
```


### Install Kubernetes Dashboard
1. kubernetes-dashboard.yaml
```yaml
kind: Deployment 
apiVersion: extensions/v1beta1 
metadata: 
  labels: 
    app: kubernetes-dashboard 
  name: kubernetes-dashboard 
  namespace: kube-system 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: kubernetes-dashboard 
  template: 
    metadata: 
      labels: 
        app: kubernetes-dashboard 
      # Comment the following annotation if Dashboard must not be deployed on master 
      annotations: 
        scheduler.alpha.kubernetes.io/tolerations: | 
          [ 
            { 
              "key": "dedicated", 
              "operator": "Equal", 
              "value": "master", 
              "effect": "NoSchedule" 
            } 
          ] 
    spec: 
      containers: 
      - name: kubernetes-dashboard 
        image: gcr.io/share/kubernetes-dashboard-amd64:v1.5.1      #默认的镜像是使用google的
        imagePullPolicy: IfNotPresent # 默认Always 
        ports: 
        - containerPort: 9090 
          protocol: TCP 
        args: 
          # Uncomment the following line to manually specify Kubernetes API server Host 
          # If not specified, Dashboard will attempt to auto discover the API server and connect 
          # to it. Uncomment only if the default does not work. 
          - --apiserver-host=http://10.0.10.10:8080    #注意这里是api的地址 
        livenessProbe: 
          httpGet: 
            path: / 
            port: 9090 
          initialDelaySeconds: 30 
          timeoutSeconds: 30 
--- 
kind: Service 
apiVersion: v1 
metadata: 
  labels: 
    app: kubernetes-dashboard 
  name: kubernetes-dashboard 
  namespace: kube-system 
spec: 
  type: NodePort 
  ports: 
  - port: 80 
    targetPort: 9090 
  selector: 
    app: kubernetes-dashboard 
```
2. 国内下载Image经常被墙，请看[移步](http://blog.csdn.net/qq_32971807/article/details/54693254)
3. 替换yaml中 - --apiserver-host=http://10.0.10.10:8080    #注意这里是api的地址 
systemctl status kube-apiserver -l to get the ip of api-server(Should be something like https://10.0.2.15:6443/swaggerapi/)
4. 创建kubernetes dashboard
```bash
kubectl creaet -f kubernetes-dashboard.yaml
kubectl get pods --all-namespaces
```
5. Open Dashboard URL [VM host ip instead of kube-apiserver ip]:8080/ui
