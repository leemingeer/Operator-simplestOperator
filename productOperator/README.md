[toc]

# 预准备，install kubebuilder

```
// download
curl -sSL https://github.com/kubernetes-sigs/kubebuilder/releases/download/v3.0.0/kubebuilder_linux_amd64 -O /usr/bin/kubebuilder

# check
# kubebuilder version
```

# kubebuilder init
```
kubebuilder init \
--domain ming.xyz \
--repo github.com/leemingeer/NodePoolOperator \
--skip-go-version-check


–-domain lailin.xyz 我们的项目的域名，crd的后缀group: apps.lailin.xyz

--repo xxx 是仓库地址，同时也是 go mode中的repo地址

```

## 变化
```
# tree
.
├── config
│   ├── default  # 一些默认配置
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager  # 部署 crd 所需的 yaml
│   │   ├── controller_manager_config.yaml
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus # 监控指标数据采集配置
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   └── rbac # # 部署所需的 rbac 授权 yaml
│       ├── auth_proxy_client_clusterrole.yaml
│       ├── auth_proxy_role_binding.yaml
│       ├── auth_proxy_role.yaml
│       ├── auth_proxy_service.yaml
│       ├── kustomization.yaml
│       ├── leader_election_role_binding.yaml
│       ├── leader_election_role.yaml
│       ├── role_binding.yaml
│       └── service_account.yaml
├── Dockerfile
├── go.mod
├── go.sum
├── hack
│   └── boilerplate.go.txt
├── main.go
├── Makefile
└── PROJECT

```

#  kubebuilder create api, define a resource
```
kubebuilder create api \
--group apps \
--version v1 \
--kind Application
```

该命令会增加api和controllers目录

## 变化
```
#       modified:   main.go
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       api/
#       config/crd/
#       config/rbac/application_editor_role.yaml
#       config/rbac/application_viewer_role.yaml
#       config/samples/
#       controllers/
```



# 实现 Controller()

## 定义CR

```
api/v1/application_types.go

// 修改<Kind>Spec字段
type ApplicationSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Product 该应用所属的产品
	Product string `json:"product,omitempty"`
}
```

## 生成CRD make manifests generate 

### 变化
```
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       config/crd/bases/
#       config/rbac/role.yaml

生成crd定义
cat config/crd/bases/apps.ming.xyz_applications.yaml

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.4.1
  creationTimestamp: null
  name: applications.apps.ming.xyz
spec:
  group: apps.ming.xyz  # GVK的时候的G
  names:
    kind: Application
    listKind: ApplicationList
  scope: Namespaced
  versions:
  - name: v1            # GVK的时候的V
    schema:
      openAPIV3Schema:
        properties:
            apiVersion: xx
            kind：
            metadata
            spec:       # GVR的时候的R
                properties:
                    product:
                        description: Product 该应用所属的产品
                    type: string
                type: object
            status:
              description: ApplicationStatus defines the observed state of Application
              type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []

```

## 实现 controller

```
// controllers/application_controller.go

func (r *ApplicationReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
        _ = r.Log.WithValues("application", req.NamespacedName)

        // your logic here
        r.Log.Info("app changed", "ns", req.Namespace)

        return ctrl.Result{}, nil
}


```
# make install 安装 CRD

# make run运行 controller

# 测试
```
kubectl apply -f config/samples/apps_v1_application.yaml

# cat config/samples/apps_v1_application.yaml
apiVersion: apps.ming.xyz/v1
kind: Application
metadata:
  name: application-sample
spec:
  product: ming

```

控制器可以查看到
```
2021-10-13T14:22:11.568+0800    INFO    controllers.Application app changed     {"ns": "default"}
```
