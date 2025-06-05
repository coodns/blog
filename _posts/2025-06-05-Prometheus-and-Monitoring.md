---
layout: post
category: k8s
---


## 1. Prometheus 설치하기

Prometheus는 시계열 데이터베이스를 기반으로 한 오픈소스 모니터링 및 알림 시스템입니다. 다양한 방법으로 설치할 수 있지만, 여기서는 Docker를 이용한 간편한 설치 방법을 소개합니다.

### Docker를 이용한 Prometheus 설치

먼저 간단한 `prometheus.yml` 설정 파일을 생성합니다:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['host.docker.internal:9090']
```

그런 다음 Docker를 사용하여 Prometheus를 실행합니다:

```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

설치가 완료되면 브라우저에서 `http://localhost:9090`으로 접속하여 Prometheus UI를 확인할 수 있습니다.

![image](https://github.com/user-attachments/assets/754340a7-0610-4d46-8d1f-800808661906)

## 2. Prometheus 소개 및 메트릭 수집

### Prometheus란?

Prometheus는 SoundCloud에서 개발된 오픈소스 시스템 모니터링 및 알림 툴킷입니다. 2016년에 Cloud Native Computing Foundation에 두 번째 호스팅 프로젝트로 합류했으며, 현재는 Kubernetes와 함께 클라우드 네이티브 환경에서 가장 널리 사용되는 모니터링 솔루션입니다.

### 주요 특징

- 다차원 데이터 모델 (시계열 데이터는 메트릭 이름과 키-값 쌍으로 식별)
- PromQL이라는 유연한 쿼리 언어
- 분산 스토리지에 의존하지 않음
- HTTP 풀 모델을 통한 시계열 데이터 수집
- 푸시 게이트웨이를 통한 단기 작업 지원
- 서비스 디스커버리 또는 정적 구성을 통한 대상 검색
- 다양한 그래프 및 대시보드 지원

### 메트릭 수집 예시

Prometheus 자체 메트릭을 수집해 보겠습니다. Prometheus UI에서 다음 쿼리를 실행해 보세요:

```
prometheus_http_requests_total
```

이 쿼리는 Prometheus 서버가 처리한 HTTP 요청의 총 수를 보여줍니다.

<Prometheus UI에서 prometheus_http_requests_total 쿼리 결과 화면>

또한 시스템 메트릭을 수집하기 위해 Node Exporter를 설치할 수 있습니다:

```bash
docker run -d \                           
  --name node-exporter \
  -p 9100:9100 \
  prom/node-exporter
```

그리고 `prometheus.yml` 파일에 다음 설정을 추가합니다:

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['host.docker.internal:9100']
```

이제 메모리 사용률과 같은 데이터를 쿼리 해보실 수 있습니다.:

```
100 * (1-(node_memory_MemFree_bytes + node_memory_Cached_bytes + node_memory_Buffers_bytes) / node_memory_MemTotal_bytes)
```

![image](https://github.com/user-attachments/assets/42a97df6-7f3e-450d-b1a4-2347b2d5b13c)

## 3. Prometheus 작동 원리

### 풀(Pull) 기반 아키텍처

Prometheus는 대부분의 모니터링 시스템과 달리 '풀(Pull)' 모델을 사용합니다. 즉, 모니터링 대상에서 데이터를 가져오는 방식으로 작동합니다. 이는 다음과 같은 이점을 제공합니다:

1. 중앙 집중식 제어 및 구성
2. 네트워크 파티션 시 더 나은 가시성
3. 여러 모니터링 시스템에서 동일한 엔드포인트 스크래핑 가능

[imnst](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FLKz03%2FbtsDs4ipr7f%2FUAO7sEP5jOLds4bRRThuzK%2Fimg.png)![image](https://github.com/user-attachments/assets/02681846-e589-43aa-9d28-dc97d6dfc85e)

### 주요 구성 요소

Prometheus 에코시스템은 다음과 같은 주요 구성 요소로 이루어져 있습니다:

1. **Prometheus 서버**: 시계열 데이터를 수집하고 저장
2. **클라이언트 라이브러리**: 애플리케이션 코드를 계측하기 위한 라이브러리
3. **Exporter**: 특정 시스템의 메트릭을 Prometheus 형식으로 노출
4. **Alertmanager**: 알림 처리
5. **Pushgateway**: 단기 작업의 메트릭 지원
6. **시각화 도구**: Grafana와 같은 도구와의 통합

## 4. Prometheus 메트릭 구성

Prometheus 메트릭은 다음과 같은 요소로 구성됩니다:

### 메트릭 구성 요소

1. **이름(Name)**: 측정하는 항목을 설명하는 이름 (예: `http_requests_total`)
2. **값(Value)**: 실제 측정된 숫자 값
3. **타임스탬프(Timestamp)**: 메트릭이 수집된 시간
4. **라벨(Labels)**: 메트릭의 차원을 정의하는 키-값 쌍 (예: `method="GET"`, `status="200"`)

### 메트릭 유형

Prometheus는 네 가지 핵심 메트릭 유형을 제공합니다:

1. Counter: 단조롭게 증가하는 값 (예: 시스템 부팅 이후 누적된 값)
```
   node_network_transmit_bytes_total{device="eth0"} 
```
  이 메트릭은 eth0 네트워크 인터페이스를 통해 전송된 총 바이트 수를 보여줍니다. 이 값은 시스템이 실행되는 동안 계속 증가합니다.

  ![image](https://github.com/user-attachments/assets/7db98be6-9633-4209-a57f-3a91d576c27d)


2. Gauge: 임의로 오르내릴 수 있는 값 (예: 현재 CPU 사용량, 메모리 사용량)
```
   node_memory_MemAvailable_bytes{instance="localhost:9100"} 
```
  이 메트릭은 현재 사용 가능한 메모리가 약 2.5GB임을 보여줍니다. 이 값은 시스템 활동에 따라 증가하거나 감소할 수 있습니다.

![image](https://github.com/user-attachments/assets/da8a0698-a25e-4d07-a9f1-e3a6cb1a3792)
  

3. Histogram: 관측값의 분포와 합계를 측정

```
   node_disk_io_time_seconds_bucket{device="sda", le="0.1"}
   node_disk_io_time_seconds_bucket{device="sda", le="0.5"} 
   node_disk_io_time_seconds_sum{device="sda"} 
   node_disk_io_time_seconds_count{device="sda"} 
```

  이 메트릭은 디스크 I/O 작업 시간의 분포를 보여줍니다. 0.1초 이하로 완료된 작업이 1,205개, 0.5초 이하로 완료된 작업이 2,183개이며, 총 2,540개 작업의 총 소요 시간은 120.7초입니다.

4. Summary: Histogram과 유사하지만 클라이언트 측에서 분위수 계산

```
   node_scrape_collector_duration_seconds{collector="filesystem", quantile="0.5"}
   node_scrape_collector_duration_seconds{collector="filesystem", quantile="0.9"}
   node_scrape_collector_duration_seconds_sum{collector="filesystem"}
   node_scrape_collector_duration_seconds_count{collector="filesystem"}
```

  이 메트릭은 파일시스템 수집기의 실행 시간 분포를 보여줍니다. 실행의 50%는 0.012초 이하, 90%는 0.025초 이하로 완료되었으며, 총 320회 실행의 총 소요 시간은 5.6초입니다.

Node Exporter는 이러한 다양한 메트릭 유형을 사용하여 CPU 사용량, 메모리 사용량, 디스크 I/O, 네트워크 트래픽 등 시스템 리소스에 대한 포괄적인 모니터링 데이터를 제공합니다. 이 데이터를 Prometheus에서 수집하고 분석
하여 시스템 성능과 상태를 모니터링할 수 있습니다.

## 5. API 서버 메트릭 수집

### API 서버 메트릭 수집 가능성

네, Python, Go, Node.js 등의 API 서버에서 생성하는 메트릭을 Prometheus가 수집할 수 있습니다. 대부분의 주요 프로그래밍 언어와 프레임워크에는 Prometheus 클라이언트 라이브러리가 있어 애플리케이션 메트릭을 쉽게 노출할 수 있습니다.

### Django 애플리케이션 메트릭 수집 예시

이 프로젝트에서는 Django 애플리케이션에 `django_prometheus` 라이브러리를 통합하여 메트릭을 수집했습니다.

1. 먼저 라이브러리를 설치합니다:
   ```bash
   pip install django-prometheus
   ```

2. Django 설정에 미들웨어와 앱을 추가합니다:
   ```python
   INSTALLED_APPS = [
       # ...
       'django_prometheus',
   ]

   MIDDLEWARE = [
       'django_prometheus.middleware.PrometheusBeforeMiddleware',
       # 기존 미들웨어...
       'django_prometheus.middleware.PrometheusAfterMiddleware',
   ]
   ```

3. URL 설정에 메트릭 엔드포인트를 추가합니다:
   ```python
   from django.urls import path
   from django_prometheus import exports

   urlpatterns = [
       # ...
       path('metrics/', exports.ExportToDjangoView, name='prometheus-metrics'),
   ]
   ```

4. Prometheus 설정에 Django 애플리케이션을 스크래핑 대상으로 추가합니다:
   ```yaml
   scrape_configs:
     - job_name: 'django'
       scrape_interval: 5s
       metrics_path: /metrics
       static_configs:
         - targets: ['django-app:8000']
   ```

## 6. prometheus.yml 설정 파일 리뷰

이 프로젝트에서 사용한 `prometheus.yml` 파일을 살펴보겠습니다:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'django'
    scrape_interval: 15s
    metrics_path: /metrics
    scheme: http
    static_configs:
      - targets: ['host.docker.internal:8000']
  - job_name: 'node'
    metrics_path: /metrics
    scrape_interval: 15s
    scheme: http
    static_configs:
      - targets: [ 'host.docker.internal:9100' ]
```

### 주요 설정 요소 설명

1. **global**: 전역 설정을 정의합니다.
   - `scrape_interval`: 대상을 스크래핑하는 기본 간격
   - `evaluation_interval`: 규칙을 평가하는 간격

2. **scrape_configs**: 스크래핑할 대상 목록을 정의합니다.
   - `job_name`: 작업 식별자
   - `scrape_interval`: 이 작업에 대한 사용자 정의 스크래핑 간격
   - `metrics_path`: 메트릭을 노출하는 HTTP 엔드포인트 경로
   - `scheme`: 사용할 프로토콜 (http 또는 https)
   - `static_configs`: 정적으로 구성된 대상 목록
     - `targets`: 스크래핑할 호스트 및 포트
   - `headers`: 요청에 포함할 HTTP 헤더

### 고급 설정 옵션

더 복잡한 환경에서는 다음과 같은 고급 설정을 추가할 수 있습니다:

1. **서비스 디스커버리**: 동적 환경에서 대상을 자동으로 발견
   ```yaml
   - job_name: 'kubernetes-pods'
     kubernetes_sd_configs:
       - role: pod
   ```

2. **릴레이블링**: 대상 메타데이터 조작
   ```yaml
   relabel_configs:
     - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
       action: keep
       regex: true
   ```

3. **알림 규칙**: 조건에 따른 알림 정의
   ```yaml
   rule_files:
     - 'alert.rules'
   ```

4. **원격 쓰기/읽기**: 장기 스토리지 통합
   ```yaml
   remote_write:
     - url: "http://remote-storage/write"
   ```
