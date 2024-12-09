# troubleshooting-k8s
Source - learnk8s.io

```mermaid
graph TD
    A[НАЧАЛО] --> B[kubectl get pods]
    B --> |Есть ли pod в PENDING?| C[Заполнен ли кластер?]
    C --> |ДА| D[Разверните более крупный кластер]
    C --> |НЕТ| E[Упираетесь ли в лимиты ResourceQuota?]
    E --> |ДА| F[Используйте лимиты ResourceQuota]
    E --> |НЕТ| G[Монтируете PersistentVolume с атрибутом Pending?]
    G --> |ДА| H[Исправьте PersistentVolumeClaim]
    G --> |НЕТ| I[Проблема на стороне планировщика]
    H --> J[Проблема на стороне kubelet]
    I --> J

    B --> |Есть ли pod в RUNNING?| K[kubectl logs pod <pod-name>]
    K --> L[Доступны логи приложения?]
    L --> |НЕТ| M[Исправьте проблему в приложении]
    L --> |ДА| N[Статус pod-а ImagePullBackOff?]
    N --> |ДА| O[Исправьте имя образа или тег]
    N --> |НЕТ| P[Статус pod-а CrashLoopBackOff?]
    P --> |ДА| Q[kubectl logs pod --previous]
    Q --> R[Исправьте ошибки в приложении]
    P --> |НЕТ| S[Pod часто перезапускается из-за ошибки в Dockerfile?]
    S --> |ДА| T[Исправьте Dockerfile]
    S --> |НЕТ| U[Обратитесь к StackOverflow]

    B --> |Есть ли pod в READY?| V[Readiness-probe завершилась ошибкой?]
    V --> |ДА| W[Исправьте readiness-probe]
    V --> |НЕТ| X[Возможная проблема с API или kubelet]

    B --> |Есть доступ к приложению?| Y[kubectl port-forward <pod-name>:8080:<pod-port>]
    Y --> Z[Подключение корректно работает?]
    Z --> |НЕТ| AA[Исправьте pod (например, измените hostPort)]
    Z --> |ДА| AB[Pod'ы корректно функционируют]

    AB --> AC[Ingress корректно функционирует]
    AB --> |Service корректно функционирует| AD

    AD --> AE[kubectl describe service <service-name>]
    AE --> |Виден ли список endpoint-ов?| AF[Подключение корректно работает]
    AF --> |НЕТ| AG[Проблема в Controller manager]

    AC --> AH[kubectl describe ingress <ingress-name>]
    AH --> |Видны ли значения serviceName и servicePort в Backend?| AI[Исправьте serviceName и servicePort в Ingress]
    AI --> AJ[Ingress корректно функционирует]

    AJ --> AK[КОНЕЦ]
    AG --> AK
    AF --> AK
```
