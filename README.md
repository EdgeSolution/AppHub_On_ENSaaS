# AppHub_On_EnSaaS
本文档适用于部署AppHub到Wise-Paas公有云或私有云平台，包括[helm方式手动部署](#helm)和[Catalog在线订阅部署](#catalog_1)两种方式。

其中helm手动部署方式又分了三种场景，不同场景可以直接参考对应章节。

场景1：新新人在没有任何环境基础的情况下使用helm包部署，可以参考章节[1. helm方式首次部署](#1)；

场景2：之前已经helm部署过，现在想通过helm包完整的更新AppHub，可以参考章节[2. helm方式升级image等配置](#2)

场景3：仅需要更新某个pod内的image，保留其他pod或者数据库的情况下，可以参考章节[3.kubectl命令直接更新image](#3)

# <span id="helm">helm方式手动部署</span>

## <span id="1">1. helm方式首次部署</span>
### 1.1 环境准备
#### (1) 安装kubectl

**参考步骤**

step1：终端运行下面命令获取kubectl的最新发行版
```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
```
step2：安装kubectl
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
```
setp3: 将kubectl加入环境变量
例如在/etc/profile末尾添加：`export PATH=.:~/.local/bin/kubectl:$PATH`
然后执行`Source /etc/profile`使其生效

setp4：最后通过`kubectl version –client`验证

#### (2) 安装helm
以Ubuntu为例，官网网站，ubuntu下安装的方式为：

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```


### 1.2 订阅服务
进入EnSaaS4.0 Management Portal页面， 点击工作台中的服务中心，如下图

![图1-进入服务中心](https://user-images.githubusercontent.com/65381865/164954623-ff168449-4096-4d47-984a-fecf7c27798b.png)

#### (1) 订阅RabbitMQ
订阅RabbitMQ服务并生成secret 

**step 1: 进入RabbitMQ服务密钥管理页面**

选择的RabbitMQ服务，点击密钥管理，进入其密钥管理页面，如下图

![图2-rabbitmq密钥管理页面](https://user-images.githubusercontent.com/65381865/164954622-da608efb-e6a1-4d69-b2ed-f0333bfe0edf.png)

**step 2: 创建RabbitMQ服务的密钥**

首先点击右上角“+”号，如图所示，开始创建

![图2-1-加号](https://user-images.githubusercontent.com/65381865/164954629-b844e94e-fffc-4b38-b2a4-fe8675a68915.png)

然后在如下图所示的弹出框中输入相应内容，即输入secret的名称，选择需要部署apphub的集群、工作空间和命名空间，点击确定，成功创建secret

![图2-2-创建密钥](https://user-images.githubusercontent.com/65381865/164954633-16ac48ec-c133-4ce1-89ac-082526d44142.png)

下图为example，可以共参考

![图2-3创建密钥demo](https://user-images.githubusercontent.com/65381865/164954639-336f3e57-b9f8-422f-9885-7ecf59ac4a9d.png)

创建成功后，在密钥管理列表中可以看到，如下图

![图2-4创建成功](https://user-images.githubusercontent.com/65381865/164954650-bea25ec3-f040-4d68-ada1-e786df4363a8.png)

#### (2) 订阅 Postgresql DB服务

**step1：进Postgresql DB其密钥管理页面**

选择的Postgresql DB服务，点击密钥管理，进入其密钥管理页面，如下图

![图3-db服务进入密钥管理页面](https://user-images.githubusercontent.com/65381865/164954660-64c24689-1281-4c50-b01a-32550307caa9.png)

**step2：创建Postgresql DB服务的密钥**

首先同RabbitMQ一样点击右上角“+”号开始创建

**注意事项**

**1. 切记secret名称必须与2.1创建的RabbitMQ服务的secret名称保持一致**

**2. 添加参数(group: g_apphub)**


举例如下图所示

![图3-1-举例demo](https://user-images.githubusercontent.com/65381865/164954675-7db8d5d4-021f-4f1c-b900-adcbb3d88917.png)

创建成功后，会在密钥管理列表中可以看到，如下图

![图3-2创建成功](https://user-images.githubusercontent.com/65381865/164954678-6115a02a-fc5b-41cb-8ef8-4bdbad587e19.png)

### 1.3. 部署
#### (1) 配置kubectl

**step1: 从Management Portal下载kubectl的confi档**

如下图所示位置，下载
![image](https://user-images.githubusercontent.com/65381865/164957973-6b6748c0-11b9-4b95-ab93-310fe8fa70de.png)


**step2：将下载的config档拷贝到本地**

将config文档拷贝到helm部署机的/root/.kube目录下，并重命名为config


#### (2) 下载AppHub的helm包

**方法1：git clone（需要git环境）**

如果需要安装最新版本，则执行如下：

```
git clone https://github.com/EdgeSolution/AppHub_On_EnSaaS.git
```

如果需要安装指定版本，例如1.0.2，则执行如下：
```
git clone --branch 1.0.2 https://github.com/EdgeSolution/AppHub_On_EnSaaS.git
```

**方法2：网页端直接下载zip包**

通过网页打开https://github.com/EdgeSolution/AppHub_On_EnSaaS.git ，选择相应的tag版本，直接下载即可。



#### (3) 修改values.yml相关项
如果是git clone方式获取helm包，则进入路径修改values.yml
如果是下载的zip/tar.gz包，则解压后，进入路径修改values.yml

```
cd AppHub_On_EnSaaS/AppHub-HelmChart
vi values.yml
```

如下图：

![图4-value截图](https://user-images.githubusercontent.com/65381865/164954684-6e184971-1930-490e-a451-d6e1fba67f11.png)

其中,

1：secret name

2-4,6：与MP域名后缀一致

5，为站点名称

7：.命名空间名字和.集群名字组合

#### (4) 执行helm install
在命令行终端AppHub-HelmChart目录，执行：
```
helm install apphub-manager –n $namespace .
```
其中，namespace为实际要部署的目标命令空间名称。

例如，本地demo部署命令如下： `helm install apphub-manager -n apphub .`


## <span id="2">2. helm方式升级image等配置</span>

### 2.1 原部署环境仍然可用

如果原有的helm安装包文件都存在，则直接修改AppHub_On_EnSaaS/AppHub-HelmChart下的values.yml中image版本tag版本号，然后升级即可

**step1：修改value.yml中image版本**

```
cd AppHub_On_EnSaaS/AppHub-HelmChart
vi values.yml
```
参考Github上最新的value.yml中相应的image tag version修改。建议不要直接覆盖value.yml，否则还需要按照首次安装方式那样修改value.yml中global:部分的相关项。
仅修改docker images版本，如下图所示：

![image](https://user-images.githubusercontent.com/65381865/164960716-0b0b5b9b-b3ad-4d5f-9be5-3c9538d8cd65.png)

**step2: helm upgrade**

```
helm upgrade apphubmanager -n apphub .
```
### 2.2 使用新的部署环境升级部署

当原来helm部署环境不存在时，需要参考部分首次部署的步骤即：
#### (1) 环境准备
参考首次安装的环境准备步骤

#### (2) helm upagrade
**step1: 配置kubectl config档，下载AppHub的helm包，修改value.yml**

参考首次安装的3.1-3.3

**step2：执行helm upgarde**

在命令行终端AppHub-HelmChart目录，执行：
```
helm upgrade apphub-manager –n $namespace .
```
其中，namespace为实际要部署的目标命令空间名称。

例如，本地demo部署命令如下： `helm upgrade apphub-manager -n apphub .`


## <span id="3">3. kubectl命令直接更新image</span>

以修改apphub-manager的image为例说明
### (1)环境准备

假设已经有kubectl环境，仅需将config档拷贝到/root/.kube目录下，并重命名为config

### (2)kubectl命令更新image
以namespace为develop为例说明

**step1：先查看当前namespace下面的pod，确认环境OK**

```
#kubectl get pods -n develop
...
apphub-manager-7d65c89b8b-jscpq                       1/1     Running   0          14m
apphub-repo-f767bd8f4-klgr2                           1/1     Running   0          170d
...
```

**step2：edit deployment配置，修改apphub-manager的image**

```
#kubectl edit deployment apphub-manager -n apphub
```
![image](https://user-images.githubusercontent.com/65381865/181678128-5522a1a4-7cc5-48e7-959c-b223f8002af3.png)

修改后:wq，保存退出

正常情况下，image会自动更新部署，我们可以通过`kubectl get pods -n develop`查看当前pod是否有创建更新。

**补充说明：**

如果有异常，例如新的apphub-manager pod的状态为pending，需要先排查原因。
例如：

```
apphub-manager-59f6f775fc-sxbm9                       1/1     Running   1          210d
apphub-manager-64d478b66-mt7fh                        0/1     Pending   0          76s
apphub-repo-f767bd8f4-klgr2                           1/1     Running   0          170d

```
(1)首先kubectl describe查看失败原因

```
kubectl describe pod apphub-manager-64d478b66-mt7fh -n develop
```
(2)然后根据原因相应修改，例如是CPU资源不足导致时，则需要修改CPU资源


# <span id="catalog_1">catalog在线订阅部署</span>

**step1：首先登录catalog主页，选择AppHub**

![catalog主页搜索AppHub](https://user-images.githubusercontent.com/65381865/168772693-6ec76baf-0dc4-439e-9c20-fb757d9d3f10.png)


**step2：订阅AppHub**

根据自己的需求进行相应订阅，这里我们以订阅免费试用版为例进行说明，点击“立即订阅”

![选择相应项订阅](https://user-images.githubusercontent.com/65381865/168772794-a9ad84d0-e0ee-4efe-b00b-d8b1191f27e2.png)

## 1. 针对没有EnSaaS环境的用户

如果您还没有企业账户，则先在Marketplace中完成注册和充值的步骤；如果您已经在Marketplace注册企业账户并完成充值后，则直接Marketplace页面选择AppHub产品直接跳转到catalog门户进行接下来的操作。

**step1：选择正确的订阅号和应用部署**

这里选择Standard Total因为EnSaaS环境资源需要付费，所以这里会产生费用。（注意选择正确的订阅号）

![首次订阅—step1](https://user-images.githubusercontent.com/65381865/168772840-2782f224-dc3f-4574-ade9-0c81307cfe25.png)


EnSaaS云服务上AppHub免费试用版支持设备在线数量最大为50，如果需要额外设备数量可以在部署配置这里勾选额外设备数进行结算，如果不需要则直接点击“下一步”。

![首次订阅—step1-部署配置选项](https://user-images.githubusercontent.com/65381865/168772891-ad2c127f-25f2-4171-b1d4-938e5b0a277f.png)


**step2：在总结页面确认**

![首次订阅—step2-总结](https://user-images.githubusercontent.com/65381865/168772953-ca734246-4842-4ea8-9abf-1143dadac1b5.png)

确认无误后会产生部署订单并如下图：
![首次订阅—step2-总结-确定后3](https://user-images.githubusercontent.com/65381865/168773054-89892d97-0368-4315-9d52-113fe80217b2.png)
部署完成后会邮件通知用户。



## 2. 针对已经有EnSaaS环境的用户

如果您已经拥有一个EnSaaS环境，只需要部署AppHub这个SRP应用，则按照如下方式操作。

**step1：选择正确的订阅号和应用部署**

这里选择Standard Single，注意选择正确的订阅号。

![已经有ensaas环境的订阅-1](https://user-images.githubusercontent.com/65381865/168773115-b0fec1f0-d694-41aa-8d92-a4ca61061cae.png)


EnSaaS云服务上AppHub免费试用版支持设备在线数量最大为3，如果需要额外设备数量可以在部署配置这里勾选额外设备数进行结算，如果不需要则直接点击“下一步”。

![已经有ensaas环境的订阅-2-additional](https://user-images.githubusercontent.com/65381865/168773138-07cc6002-2b48-49b2-bcd7-a011ba0c2460.png)


**step2：k8s服务和基础服务**

K8S服务选择相应正确项，基础服务有PostgreSQL和RabbitMQ。选择完成后点击下一步，如下图。
![已经有ensaas环境的订阅-3-k8s服务](https://user-images.githubusercontent.com/65381865/168773167-4e20a491-18ed-47bb-b507-c0c48220e9eb.png)
![已经有ensaas环境的订阅-3-基础服务](https://user-images.githubusercontent.com/65381865/168773175-8b40617a-0b81-463d-8b9d-2e348ba7abf1.png)


**step3：总结，确认订单**

在总结页面确认各项内容正确后点击确认，如下图：

![已经有ensaas环境的订阅-4总结](https://user-images.githubusercontent.com/65381865/168773204-a753bf63-b9ae-43b9-b1be-f01274e8d270.png)

确认后，在订单页面可以看到我们的订单详情，等待订单完成。

**step4：获取AppHub portal**

订单完成后，登录managerment portal，在应用页面就可以看到刚刚部署的AppHub信息，获取访问url，如下图所示。

![image](https://user-images.githubusercontent.com/65381865/169188182-adc52b5c-c664-4ec5-8635-ba4832d4d132.png)


# catalog在线订阅license并激活

当用户需要更多在线设备连接AppHub进行远程控制和远程部署等操作时，则需要购买license激活。购买license也是在catalog上进行。
## 1. EnSaaS平台上AppHub Manager版本激活
以下截图是在测试平台上测试使用截图，实际购买操作环境基本一致。

### 1.1 订阅license并激活

**step1：首先登录catalog主页，选择AppHub**

![catalog主页搜索AppHub](https://user-images.githubusercontent.com/65381865/168772693-6ec76baf-0dc4-439e-9c20-fb757d9d3f10.png)

**step2：订阅License for AppHub**

这里我们以购买一个月设备数量100台的场景为例进行购买。

（1）选择正确的订阅号

（2）点击选择确认“Monthly License for AppHub”

（3）勾选“Additional”，数量选择“2” （以50台设备为单位，数量选择2，即2x50=100）

![image](https://user-images.githubusercontent.com/65381865/168939944-bc296f19-2766-4d39-931a-bff9392eaa41.png)

![image](https://user-images.githubusercontent.com/65381865/168939502-1b4f9853-33a0-431a-9d97-d63159a34488.png)

**step3：总结确认账单**

![image](https://user-images.githubusercontent.com/65381865/168940075-3311602c-5d37-4111-9606-7bed73c60679.png)

等待账单完成后，在订单页面可以看到如下截图：

![image](https://user-images.githubusercontent.com/65381865/168940520-5875ced5-2fc2-4847-9e75-ce028369230a.png)



## 2. 本地AppHub Manager版本激活
这里补充说明一下，加入在本地部署了一个AppHub Manager server，想通过catalog购买license激活的使用方法
### 2.1 在线订阅license

关于AppHub manager的license我们分按月购买和买断两种方式，每种方式都支持选择设备数量，根据时间和设备数量的不通有不通价位。


**step1：首先登录catalog主页，选择AppHub**

![catalog主页搜索AppHub](https://user-images.githubusercontent.com/65381865/168772693-6ec76baf-0dc4-439e-9c20-fb757d9d3f10.png)


**step2：订阅License for AppHub**

这里我们以购买一个月设备数量100台的场景为例进行购买。

（1）选择正确的订阅号

（2）点击选择确认“Monthly License for AppHub”

（3）勾选“Additional”，数量选择“2” （以50台设备为单位，数量选择2，即2x50=100）

![image](https://user-images.githubusercontent.com/65381865/168939944-bc296f19-2766-4d39-931a-bff9392eaa41.png)

![image](https://user-images.githubusercontent.com/65381865/168939502-1b4f9853-33a0-431a-9d97-d63159a34488.png)


**step3：总结确认账单**

![image](https://user-images.githubusercontent.com/65381865/168940075-3311602c-5d37-4111-9606-7bed73c60679.png)

等待账单完成后，在订单页面可以看到如下截图：

![image](https://user-images.githubusercontent.com/65381865/168940520-5875ced5-2fc2-4847-9e75-ce028369230a.png)

### 2.2 license激活

**step1：激活license

在导航栏选择“许可证”，
![image](https://user-images.githubusercontent.com/65381865/168940676-d35f0da6-b73e-4886-ae70-b41d9a734630.png)

选择订单对应的“激活”

![image](https://user-images.githubusercontent.com/65381865/168940749-7ad45e92-3f85-4dad-9c9f-034014fe830a.png)

在AppHub Manager的license页面有workspace ID和Mac地址信息，复制后在弹出对话框中复制后点击确认即可激活

**step2：下载许可证

license激活后，在许可证页面点击“许可证文件”下载许可证书，然后在本地server上传激活



