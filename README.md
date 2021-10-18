# MSA-Platform

----

- [MSA-Platform](#msa-platform)
  - [AS-IS](#as-is)
    - [~~Install Istio with Helm~~](#install-istio-with-helm)
    - [Install Istio with Operator](#install-istio-with-operator)
      - [Install istioctl](#install-istioctl)
      - [Deploy the Istio operator](#deploy-the-istio-operator)
      - [Install Istio with the operator](#install-istio-with-the-operator)
    - [Install Keycloak with Helm](#install-keycloak-with-helm)

----

## AS-IS

Helm 을 이용한 구성

- istio-gateway
- odic-proxy
- keycloak
- prometheus / grafana

1. K8s에 Helm 으로 구성
2. 필요한 정보 구성
3. 전체 연동
4. 이를 바탕으로 platform 화에 대한 설계 및 개발

시나리오 수동 구성과 platform 의 자동 구성 차이를 설명하자.

- 설치하는 시나리오
- 삭제하는 시나리오

### ~~Install Istio with Helm~~

:hand: Istio 에서 Operator 를 제공하므로 Operator로 해보자. [Istio Operator Install](#istio-operator-install)

> - 1.11.4 버전 활용
> - https://istio.io/latest/docs/setup/install/helm/

```sh
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.11.4 TARGET_ARCH=x86_64 sh -
```

### Install Istio with Operator

> - https://istio.io/latest/docs/setup/install/operator/

#### Install istioctl

> - https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/

```sh
curl -sL https://istio.io/downloadIstioctl | sh -
```

Add the istioctl client to your path, on a macOS or Linux system:

```sh
export PATH=$PATH:$HOME/.istioctl/bin
```

#### Deploy the Istio operator

```sh
istioctl operator init
```

#### Install Istio with the operator

```sh
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istiocontrolplane
spec:
  profile: default
EOF
```

> profile 을 데모로 하면 `istiod` 와 `istio-ingressgateway` 를 설치한다. 자세한 정보는 여기 [config profiles](https://istio.io/latest/docs/setup/additional-setup/config-profiles/)

|                        | default | demo | minimal | external | empty | preview |
|------------------------|---------|------|---------|----------|-------|---------|
| Core components        |         |      |         |          |       |         |
| - istio-egressgateway  |         | ✔   |         |          |       |
| - istio-ingressgateway | ✔      | ✔    |         |          |       | ✔      |
| - istiod               | ✔      | ✔    | ✔      |          |       | ✔       |

관련된 pod 조회

```console
$ kubectl get po -A
NAMESPACE        NAME                                      READY   STATUS    RESTARTS   AGE
istio-operator   istio-operator-6f9dcd4469-w4n45           1/1     Running   0          4h45m
istio-system     istiod-59b7bcdb74-c4gtn                   1/1     Running   0          4h34m
istio-system     svclb-istio-ingressgateway-kxlnr          3/3     Running   0          4h34m
istio-system     istio-ingressgateway-8dbb57f65-2qx6z      1/1     Running   0          4h34m
```

### Install Keycloak with Helm

> - https://github.com/codecentric/helm-charts/tree/master/charts/keycloak (GDC에서 사용)
> - ~~https://bitnami.com/stack/keycloak/helm~~

```sh
helm repo add codecentric https://codecentric.github.io/helm-charts
```

```sh
helm install keycloak codecentric/keycloak
```

keycloak 가 모두 실행 된 후에 아래 명령으로 port-forwarding 을 통해서 접근할 수 있다.

```sh
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=keycloak,app.kubernetes.io/instance=keycloak" -o name)
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace default port-forward "$POD_NAME" 8080
```

> - 음..근데 id / password 가 뭐지?
> - 접속해 보면 홈에서 계정 생성을 바로 할 수 있다. 여기서 생성하면 로그인이 가능하다.
