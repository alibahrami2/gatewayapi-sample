# Gateway API canary

Ingress solved getting HTTP into the cluster. It never solved who owns the edge.

That’s how you end up with platform policy and app routing in one object, and the real behavior in `nginx.ingress.kubernetes.io/*`. Those annotations are not a spec. They are the controller.

Gateway API splits it.

- **Platform** owns the `Gateway`: listeners, TLS, ports, which namespaces may attach.
- **App teams** own the `HTTPRoute`: hostname, matches, backends, weights.

## Scenario

New hostname. 10% of traffic to v2.

Platform adds one Gateway — HTTP listener, hostname `shop.example.com`, `allowedRoutes` only from the same namespace. The app repo only has the HTTPRoute: `parentRef` to that Gateway, the hostname, two `backendRefs`.

| Backend | Weight |
| --- | --- |
| `shop-v1` | 90 |
| `shop-v2` | 10 |

Canary is a field. Nobody opens ingress-controller docs. Platform doesn’t touch the route again.

Weights are relative. `90` and `10` is a 90/10 split.

Hostname on the listener and the HTTPRoute must overlap, or the route never attaches.

## Apply

Needs Gateway API on the cluster and a `GatewayClass` named `cilium`.

```bash
kubectl apply -f namespace.yaml -f gateway.yaml -f service.yaml -f httproute.yaml
```

```bash
kubectl -n shop get gateway shop-gateway
kubectl -n shop get httproute shop
```

`Accepted` on the Gateway and `ResolvedRefs` on the HTTPRoute mean the route is attached.

## Files

| File | Role |
| --- | --- |
| `namespace.yaml` | `shop` |
| `gateway.yaml` | platform — listener on `8081` for `shop.example.com` |
| `service.yaml` | `shop-v1`, `shop-v2` |
| `httproute.yaml` | app — `parentRef`, weights `90` / `10` |

---

Ingress فقط این را حل کرد که HTTP چطور وارد کلاستر شود. اینکه لبهٔ شبکه مال کیست را حل نکرد.

برای همین policy پلتفرم و روت اپ می‌روند داخل یک آبجکت، رفتار واقعی هم داخل `nginx.ingress.kubernetes.io/*`. آن annotationها spec نیستند. خودِ کنترلرند.

Gateway API این را جدا می‌کند.

- پلتفرم مالک `Gateway` است: listener، TLS، پورت، کدام namespace حق attach دارد.
- تیم اپ مالک `HTTPRoute` است: hostname، match، backend، وزن.

**سناریو:** یک hostname جدید، ۱۰٪ کناری روی v2.

پلتفرم یک Gateway می‌گذارد. ریپوی اپ فقط HTTPRoute دارد — `parentRef` به همان Gateway، hostname، دو `backendRef` (`v1` برابر ۹۰، `v2` برابر ۱۰). کناری یک فیلد است. کسی داک کنترلر را باز نمی‌کند. پلتفرم دیگر به روت دست نمی‌زند.

وزن‌ها نسبی‌اند. hostname روی listener و HTTPRoute باید یکی باشد، وگرنه روت attach نمی‌شود.

```bash
kubectl apply -f namespace.yaml -f gateway.yaml -f service.yaml -f httproute.yaml
```

`Accepted` روی Gateway و `ResolvedRefs` روی HTTPRoute یعنی روت وصل شده.
