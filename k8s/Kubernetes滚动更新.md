Deployment对象可以定义一个副本集（ReplicaSet），并且支持滚动更新。具体来说，滚动更新会先在新的ReplicaSet中启动一些Pod，然后逐步停止旧的ReplicaSet中的Pod，直到所有的Pod都被更新完成。

以下是一个示例Deployment配置文件：

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
在这个示例中，spec.replicas指定了要启动的Pod的数量，spec.selector指定了要选择的Pod的标签，spec.template定义了要创建的Pod的配置，spec.strategy指定了滚动更新的策略。

spec.strategy.type指定了滚动更新的类型，这里使用了RollingUpdate类型。spec.strategy.rollingUpdate.maxUnavailable指定了在滚动更新期间最多可以停止的Pod的数量，spec.strategy.rollingUpdate.maxSurge指定了在滚动更新期间最多可以启动的Pod的数量。这两个参数的值可以根据实际情况进行调整。

当您需要进行滚动更新时，只需要更新Deployment配置文件中的spec.template部分即可。Kubernetes将自动根据您的配置进行滚动更新。