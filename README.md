# Gateway API canary sample

Ingress puts the front door and the route in one object. Gateway API splits them.

| Who | Object | Owns |
| --- | --- | --- |
| Platform | `Gateway` | listener, port, hostname, who may attach |
| App | `HTTPRoute` | path match, backends, weights |

This sample sends **90%** of traffic to `shop-v1` and **10%** to `shop-v2`.

## Apply

Needs a cluster with Gateway API and a `GatewayClass` named `cilium`.

```bash
kubectl apply -f namespace.yaml -f gateway.yaml -f service.yaml -f httproute.yaml
```

```bash
kubectl -n shop get gateway shop-gateway
kubectl -n shop get httproute shop
```

`Accepted` on the Gateway and `ResolvedRefs` on the HTTPRoute mean the route is attached. Hostname on the listener and the HTTPRoute must overlap, or nothing attaches.

## Files

- `namespace.yaml` — `shop`
- `gateway.yaml` — HTTP listener on port `8081` for `shop.example.com`
- `service.yaml` — `shop-v1`, `shop-v2`
- `httproute.yaml` — `parentRef` to the Gateway, weights `90` / `10`

Weights are relative. `90` and `10` is a 90/10 canary.

---

Ingress درِ ورودی و روت را در یک آبجکت می‌گذارد. Gateway API این را جدا می‌کند.

پلتفرم مالک `Gateway` است (listener، پورت، hostname، چه کسی حق attach دارد).  
تیم اپ مالک `HTTPRoute` است (path، backend، وزن).

این نمونه **۹۰٪** ترافیک را به `shop-v1` و **۱۰٪** را به `shop-v2` می‌فرستد.

```bash
kubectl apply -f namespace.yaml -f gateway.yaml -f service.yaml -f httproute.yaml
```

`Accepted` روی Gateway و `ResolvedRefs` روی HTTPRoute یعنی روت وصل شده. hostname روی listener و HTTPRoute باید یکی باشد، وگرنه روت attach نمی‌شود.
