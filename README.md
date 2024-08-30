<h1>Deploy do Heimdall no Cluster Kubernetes</h1>

<p>Este guia descreve como implantar o Heimdall em um cluster Kubernetes com k3s, utilizando um Deployment, Service e Persistent Volume Claim (PVC).</p>

<h2>Índice</h2>
<ul>
    <li><a href="#pré-requisitos">Pré-requisitos</a></li>
    <li><a href="#arquitetura-do-cluster">Arquitetura do Cluster</a></li>
    <li><a href="#configuração-do-heimdall">Configuração do Heimdall</a></li>
    <li><a href="#instalação-do-k3s">Instalação do k3s</a></li>
    <li><a href="#configuração-do-flannel">Configuração do Flannel</a></li>
    <li><a href="#configuração-do-metallb">Configuração do MetalLB</a></li>
    <li><a href="#deploy-de-aplicações">Deploy de Aplicações</a></li>
    <li><a href="#monitoramento-e-acesso-ao-cluster">Monitoramento e Acesso ao Cluster</a></li>
    <li><a href="#recursos-adicionais">Recursos Adicionais</a></li>
    <li><a href="#contribuições">Contribuições</a></li>
    <li><a href="#licença">Licença</a></li>
</ul>

<h2 id="pré-requisitos">Pré-requisitos</h2>
<ul>
    <li>3 Raspberry Pi 4 com pelo menos 2GB de RAM.</li>
    <li>Cartões microSD com pelo menos 16GB.</li>
    <li>Sistema operacional: Raspberry Pi OS Lite ou Ubuntu Server.</li>
    <li>Rede interna com DHCP configurado.</li>
</ul>

<h2 id="arquitetura-do-cluster">Arquitetura do Cluster</h2>
<ul>
    <li><strong>Master Node:</strong> 1 Raspberry Pi (com k3s server).</li>
    <li><strong>Worker Nodes:</strong> 2 Raspberry Pi (com k3s agent).</li>
    <li><strong>Rede:</strong> Flannel como CNI (Container Network Interface).</li>
    <li><strong>LoadBalancer:</strong> MetalLB configurado em modo Layer 2.</li>
</ul>

<h2 id="configuração-do-heimdall">Configuração do Heindall</h2>
<ol>
   <h3>1. Arquivo de Deployment</h3>
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

   <h3>2. Arquivo de Persistent Volume (PV)</h3>
   <p>Crie um arquivo YAML chamado <code>heimdall-pv.yaml</code> com o seguinte conteúdo:</p>

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

  
  <h3>3. Arquivo de Persistent Volume Claim (PVC)</h3>
   <p>Crie um arquivo YAML chamado <code>heimdall-pvc.yaml</code> com o seguinte conteúdo:</p>

  <pre><code>apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: heimdall-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 2Gi
    storageClassName: standard
    </code></pre>

   
   <h3>4. Arquivo de Service</h3>
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


 <h3>5. Aplicando os arquivos ao cluster</h3>
  <p>Execute os seguintes comandos para aplicar os arquivos YAML ao cluster Kubernetes:</p>
  
  <pre><code>kubectl apply -f heimdall-deployment.yaml
    kubectl apply -f heimdall-service.yaml
    kubectl apply -f heimdall-pvc.yaml
  </code></pre>
  
<h3>6. Acessando o Heimdall</h3>
  <p>Após a implantação, você poderá acessar o Heimdall através do IP alocado pelo MetalLB:</p>
  
  <pre><code>http://&lt;IP-externo-do-Heimdall&gt;</code></pre>
  
 <h3>7. Conclusão</h3>
  <p>Agora o Heimdall está em execução no seu cluster Kubernetes, pronto para ser usado como seu painel inicial de aplicativos!</p>


  
</ol>   

