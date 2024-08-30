<h1>Deploy do Heimdall no Cluster Kubernetes</h1>

<p>Este guia descreve como implantar o Heimdall em um cluster Kubernetes com k3s, utilizando um Deployment, Service e Persistent Volume Claim (PVC).</p>

<h2>1. Arquivo de Deployment</h2>
<p>Crie um arquivo YAML chamado <code>heimdall-deployment.yaml</code> com o seguinte conteúdo:</p>

<pre><code>apiVersion: apps/v1
kind: Deployment
metadata:
  name: heimdall-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: heimdall
  template:
    metadata:
      labels:
        app: heimdall
    spec:
      containers:
      - name: heimdall
        image: linuxserver/heimdall
        ports:
        - containerPort: 80
        env:
        - name: TZ
          value: "America/Sao_Paulo"
        volumeMounts:
        - mountPath: /config
          name: config-volume
      volumes:
      - name: config-volume
        persistentVolumeClaim:
          claimName: heimdall-pvc
</code></pre>

<h2>2. Arquivo de Service</h2>
<p>Crie um arquivo YAML chamado <code>heimdall-service.yaml</code> com o seguinte conteúdo:</p>

<pre><code>apiVersion: v1
kind: Service
metadata:
  name: heimdall
spec:
  selector:
    app: heimdall
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
</code></pre>

<h2>3. Arquivo de Persistent Volume (PV)</h2>
<p>Crie um arquivo YAML chamado <code>heimdall-pvc.yaml</code> com o seguinte conteúdo:</p>

<pre><code>apiVersion: v1
kind: PersistentVolume
metadata:
  name: heimdall-pv
  labels:
    app: heimdall
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data/heimdall
</code></pre>

<h2>4. Aplicando os arquivos ao cluster</h2>
<p>Execute os seguintes comandos para aplicar os arquivos YAML ao cluster Kubernetes:</p>

<pre><code>kubectl apply -f heimdall-deployment.yaml
kubectl apply -f heimdall-service.yaml
kubectl apply -f heimdall-pvc.yaml</code></pre>

<h2>5. Acessando o Heimdall</h2>
<p>Após a implantação, você poderá acessar o Heimdall através do IP alocado pelo MetalLB:</p>

<pre><code>http://&lt;IP-externo-do-Heimdall&gt;</code></pre>

<h2>6. Conclusão</h2>
<p>Agora o Heimdall está em execução no seu cluster Kubernetes, pronto para ser usado como seu painel inicial de aplicativos!</p>
