### 环境:centos7.4/71-kmaster/72-knode/73-knode2
#### 参照文档搭建k8sV1.10.3集群

### 安装istio0.8.0
#### 下载
```
curl -L https://git.io/getLatestIstio | sh -
```
#### 进入目录
```
cd istio-0.8.0
```
#### 将 istioctl 客户端二进制文件加到 PATH 中
```
echo 'export PATH=/root/istio-0.8.0/bin:$PATH' >> /etc/profile
source /etc/profile
```
#### 确认
```
istioctl version
```
#### 安装 Istio 的核心部分/安装 Istio 的时候不启用 sidecar 之间的 TLS 双向认
```
kubectl apply -f install/kubernetes/istio-demo.yaml
```
#### 查看服务是否启动成功
```
kubectl get svc -n istio-system
kubectl get pods -n istio-system
```
#### 对指定的namespace开启自动注入sidecar
```
kubectl get ns
kubectl label namespace default istio-injection=enabled
kubectl get namespace -L istio-injection
```

#### 部署实例应用bookinfo
##### 使用 自动注入 sidecar 的方式部署的集群（自动注入失败）
```
kubectl apply -f samples/bookinfo/kube/bookinfo.yaml
```
##### 使用手动的方式注入
```
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
```
##### 创建网关
```
istioctl create -f samples/bookinfo/routing/bookinfo-gateway.yaml
```
##### 确认所有服务和 pod 已正确定义并运行
```
kubectl get services
kubectl get pods
kubectl get gateway
kubectl get virtualservice
```
##### 查看网关信息
```
kubectl get svc istio-ingressgateway -n istio-system
```
##### 启动443端口需要证书（暂时未用到，后面进行测试）
```
kubectl apply -f samples/httpbin/httpbin.yaml
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=httpbin.example.com"
```
##### 声明变量 一个是80端口 一个是443端口 
```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
```
##### 声明工作节点ip
```
export INGRESS_HOST=192.168.1.71
```
##### 网关url
```
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
echo $GATEWAY_URL   #输出：192.168.1.71:31380
export GATEWAY_URL=192.168.1.71:31380
```

##### 测试 
```
curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
```
>返回 200 代表正常

##### 浏览器访问：
http://192.168.1.72:31380/productpage   不断刷新，会有不同的页面出现
-------------------------------------------------------------------------------
##### 配置路由访问规则
```
istioctl create -f samples/bookinfo/routing/route-rule-all-v1.yaml
```
##### 清除规则
```
istioctl delete -f samples/bookinfo/routing/route-rule-all-v1.yaml
--------------------------------------------------------------------------------
#卸载bookinfo
samples/bookinfo/kube/cleanup.sh
#卸载istio
kubectl delete -f install/kubernetes/istio-demo.yaml
```