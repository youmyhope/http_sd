# HTTP SERVICE DISCOVERY

HTTP Service Discovery là công cụ để Prom dùng để bắt các target thông qua HTTP endpoint, thay vì dùng File-based Service Discover (File SD) thông qua file.

## So sánh sự khác nhau giữa HTTP SD và File-based SD

Khác biệt đầu tiên thể hiện rõ ở tên: 
*HTTP SD* sử dụng các *endpoint* và giao thức **HTTP/HTTPS** để bắt các target. Trong khi đó, *File SD* sử dụng **local file**.

| Loại          | File-based SD | HTTP SD |
| ------------- | ------------- | ------------- |
| Theo sự kiện  | Có hỗ trợ, thông qua inotify  | Không hỗ trợ |
| Tần suất update  | Ngay lập tức (nhờ inotify)  | Sử dụng refresh_interval trong HTTP header (nói chi tiết hơn ở phần sau) |
| Format  | YAML hoặc JSON  | JSON |
| Sử dụng  | Local file  | HTTP/HTTPS request |
| Bảo mật  | Các cơ chế bảo mật của file  | Cách cơ chế bảo mật của HTTP/HTTPS: TLS/SSL, OAuth 2, ... |

## Yêu cầu đối với endpoint của HTTP SD

1. Trước hết, vì HTTP SD chỉ dùng JSON, nên response trả về buộc phải chứa header `Content-Type: application/json`. 
Đồng thời body của response của phải là một *valid JSON*.

2. Trong trường hợp không có target này thì cần phải trả về một array rỗng (`[]`).

3. Như đã nói ở phần trên, về tần suất update thì sẽ do setting của refresh_interval của Prom (mặc định là 1p). 
Mỗi lần gọi request, Prom sẽ chèn một header `X-Prometheus-Refresh-Interval-Seconds` cho biết refresh interval.

## Format cho HTTP SD

Đây là format JSON HTTP SD trả về

### Format

```
[
  {
    "targets": [ "<host>", ... ],
    "labels": {
      "<labelname>": "<labelvalue>", ...
    }
  },
  ...
]
```

### Ví dụ

```
[
    {
        "targets": ["10.0.10.2:9100", "10.0.10.3:9100", "10.0.10.4:9100", "10.0.10.5:9100"],
        "labels": {
            "__meta_datacenter": "london",
            "__meta_prometheus_job": "node"
        }
    },
    {
        "targets": ["10.0.40.2:9100", "10.0.40.3:9100"],
        "labels": {
            "__meta_datacenter": "london",
            "__meta_prometheus_job": "alertmanager"
        }
    },
    {
        "targets": ["10.0.40.2:9093", "10.0.40.3:9093"],
        "labels": {
            "__meta_datacenter": "newyork",
            "__meta_prometheus_job": "alertmanager"
        }
    }
]
```
