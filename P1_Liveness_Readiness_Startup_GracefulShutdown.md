1. Configure Liveness, Readiness and Startup Probes
link: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

2. Kubernetes best practices: terminating with grace
Link: https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-terminating-with-grace

3. Health checks in ASP.NET Core
Link: https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-9.0

# 🩺 HealthCheck, Liveness & Readiness 

## 1. Mục tiêu

- **HealthCheck** kiểm tra “sức khỏe” dịch vụ (cho load balancer/Kubernetes).
- Phân biệt rõ **liveness** (app/process còn sống) và **readiness** (dịch vụ sẵn sàng nhận request).

---

## 2. Cách triển khai (Best Practice)

- **Đăng ký health check vào DI container** (giống như bỏ “đồ kiểm tra sức khỏe” vào tủ đồ dùng chung của app).
- **Gắn tag (nhãn)** cho từng health check:
    - `"live"` cho liveness probe.
    - `"ready"` cho readiness probe.

```csharp
services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: new[] { "live" })
    .AddCheck<ReadinessHealthCheck>("readiness_check", tags: new[] { "ready" })
    .AddSqlServer(config.GetConnectionString("DefaultConnection"), name: "sql", tags: new[] { "ready" })
    .AddRedis(config["Redis:Configuration"], name: "redis", tags: new[] { "ready" });

```
## 3. Cấu hình endpoint HealthCheck

- **Map healthcheck vào pipeline (middleware, không cần controller)**:
```csharp
app.MapHealthChecks("/healthz", new HealthCheckOptions
{
    Predicate = reg => reg.Tags.Contains("live")
});
app.MapHealthChecks("/readyz", new HealthCheckOptions
{
    Predicate = reg => reg.Tags.Contains("ready")
});

```
Chỉ những check có đúng tag mới được chạy ở từng endpoint.

## 4. Nguyên lý hoạt động
- DI container = Tủ đồ dùng chung cho app.
- HealthCheck = Đồ kiểm tra sức khỏe, đăng ký vào tủ.
- Tag = Nhãn dán lên từng món đồ, phân biệt mục đích kiểm tra (live/ready).
- Predicate = Bộ lọc, giúp middleware lấy đúng đồ ra kiểm tra khi có request.

## 5. Luồng thực tế
- Khi request /healthz → Middleware chỉ chạy các health check có tag "live" (kiểm tra app còn sống, cực nhẹ, không check dependency bên ngoài).
- Khi request /readyz → Middleware chỉ chạy các health check có tag "ready" (kiểm tra app, DB, Redis, trạng thái shutdown...).
- Kết quả tổng hợp trả về HTTP 200 (Healthy) hoặc 503 (Unhealthy).

## 6. Vì sao phải làm vậy?
- Đảm bảo liveness probe không làm pod restart liên tục khi DB, Redis gặp sự cố tạm thời.
- Đảm bảo readiness probe loại pod khỏi load balancer khi chưa sẵn sàng nhận request (app chưa init xong, đang shutdown, mất DB...).
- Dễ maintain, thêm/bớt check không ảnh hưởng các phần khác.

## 7. Áp dụng cho Kubernetes
````yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 80

readinessProbe:
  httpGet:
    path: /readyz
    port: 80
````
## 8. Kết luận
- HealthCheck dùng DI + Tag + Middleware để kiểm soát kỹ lưỡng liveness/readiness.
- Phân biệt rõ mục đích từng check, giúp hệ thống ổn định, dễ vận hành, đúng chuẩn cloud-native.