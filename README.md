# troubleshooting-k8s
Source - learnk8s.io

```mermaid
flowchart TD

    %% Определение стилей для различных типов узлов (по желанию)
    classDef step fill=#f9f9f9,stroke=#333,stroke-width=1px
    classDef decision fill=#ffe599,stroke=#333,stroke-width=1px
    classDef command fill=#d5e8d4,stroke=#333,stroke-width=1px
    classDef end fill=#d5e8d4,stroke=#333,stroke-width=1px

    %% Начало
    A([НАЧАЛО]):::step --> B[kubectl get pods]:::command

    %% Ветка PENDING
    B --> C{Есть ли pod'ы в PENDING?}:::decision
    C -->|ДА| D{Заполнен ли кластер?}:::decision
    C -->|НЕТ| E{Есть ли pod'ы в RUNNING?}:::decision

    D -->|ДА| F[Разверните более крупный кластер]:::step
    D -->|НЕТ| G{Упираетесь в ResourceQuota?}:::decision

    G -->|ДА| H[Используйте лимиты ResourceQuota]:::step
    G -->|НЕТ| I{Монтируете PersistentVolumeClaim\nи статус PENDING?}:::decision

    I -->|ДА| J[Исправьте PersistentVolumeClaim]:::step
    I -->|НЕТ| K{Проблема на стороне Kubelet?}:::decision

    K -->|ДА| L[Верните проблему c планировщиком]:::step
    K -->|НЕТ| M{Статус pod'a = RunContainerError?}:::decision

    M -->|ДА| N[Обратитесь к StackOverflow]:::step
    M -->|НЕТ| O[Возможно проблема с образом или CRI на Kubelet]:::step

    %% Ветка RUNNING
    E -->|ДА| P[kubectl logs pod <pod-name>]:::command
    E -->|НЕТ| Q{Есть ли pod'ы в READY?}:::decision

    %% Логи для RUNNING
    P --> R{Доступны логи приложения?}:::decision
    R -->|ДА| S[Ищите проблему в приложении]:::step
    R -->|НЕТ| T{Контейнер умирает слишком быстро?}:::decision

    T -->|ДА| U[kubectl logs pod <pod-name> --previous]:::command
    T -->|НЕТ| V[kubectl describe pod <pod-name>]:::command

    U --> W{Образ imagePullBackOff?}:::decision
    W -->|ДА| X{У образа правильное название?}:::decision
    W -->|НЕТ| Y{Статус pod'a = CrashLoopBackOff?}:::decision

    X -->|ДА| Z{Исправьте тег}:::step
    X -->|НЕТ| AA{Исправьте название образа}:::step

    Y -->|ДА| AB{Вы вычистили секреты imagePullSecrets?}:::decision
    Y -->|НЕТ| AC{Забыли указать imagePullPolicy?}:::decision

    AB -->|ДА| AD[Исправьте настройки секретов]:::step
    AB -->|НЕТ| AE[Исправьте imagePullPolicy]:::step

    AC -->|ДА| AE[Исправьте imagePullPolicy]:::step
    AC -->|НЕТ| AF[Под часто рестартуется? Проверьте LivenessProbe]:::step

    Z --> AJ[Поды корректно функционируют]:::end
    AA --> AJ
    AD --> AJ
    AE --> AJ
    AF --> AJ

    U -->|НЕТ| V

    %% Перейдём к READY
    Q -->|ДА| AG[kubectl describe pod <pod-name>]:::command
    Q -->|НЕТ| AH{ReadinessProbe завершилась ошибкой?}:::decision

    AG --> AI{Получаете неподходящую readiness-probe?}:::decision
    AI -->|ДА| AJ[Исправьте Readiness Probe]:::step
    AI -->|НЕТ| O[Возможная проблема с CRI или Kubelet]:::step

    AH -->|ДА| AJ[Исправьте Readiness Probe]:::step
    AH -->|НЕТ| O[Возможная проблема с CRI или Kubelet]:::step

    %% Проверка доступа к приложению через port-forward
    AJ --> AK{Есть доступ к приложению?}:::decision
    AK -->|НЕТ| AL{Контейнер корректно проброшен и слушает 0.0.0.0?}:::decision
    AK -->|ДА| AM[Нештатное состояние]:::step

    AL -->|ДА| AJ[Поды корректно функционируют]:::end
    AL -->|НЕТ| AM[Нештатное состояние]:::step

    %% Проверка Ingress
    AJ --> AN[kubectl describe ingress <ingress-name>]:::command
    AN --> AO{Виден ли Backend в списке правил Ingress?}:::decision

    AO -->|ДА| AP[kubectl port-forward ingress-pod <name> 8080:<ingress-port>]:::command
    AO -->|НЕТ| AQ{Значения serviceName и servicePort в Ingress соответствуют Service?}:::decision

    AQ -->|ДА| AR[Проблема скорее всего в Ingress-контроллере или доступности]:::step
    AQ -->|НЕТ| AS[Исправьте serviceName и servicePort в Ingress]:::step

    AP --> AT{Получается зайти на приложение?}:::decision
    AT -->|ДА| AU[Ingress корректно функционирует]:::end
    AT -->|НЕТ| AR[Проблема скорее всего в Ingress-контроллере или доступности]:::step

    %% Проверка Service
    AJ --> AV[kubectl describe service <service-name>]:::command
    AV --> AW{Виден ли список endpoints?}:::decision

    AW -->|ДА| AX[kubectl port-forward service/<service-name> 8080:<service-port>]:::command
    AW -->|НЕТ| AY{В спектре ли выбран namespace? Видны ли поды?}:::decision

    AY -->|ДА| AZ[Попытайтесь исправить селекторы в Service, чтобы они соответствовали Label пода]:::step
    AY -->|НЕТ| BA{Поды на неверном IP-адресе?}:::decision

    BA -->|ДА| BB[Проблема на стороне Controller manager]:::step
    BA -->|НЕТ| BC{Проблема на стороне kubelet?}:::decision

    BC -->|ДА| BD[Исправьте конфигурацию kubelet]:::step
    BC -->|НЕТ| BE[Проблема в kube-proxy]:::step

    AX --> BF{Получается зайти на приложение?}:::decision
    BF -->|ДА| BG{targetPort у Service соответствует порту в поде?}:::decision
    BF -->|НЕТ| BH[Исправьте targetPort и соответствие портов]:::step

    BG -->|ДА| BI[Service корректно функционирует]:::end
    BG -->|НЕТ| BH[Исправьте targetPort и соответствие портов]:::step

    BH --> BI

    %% Завершение схемы
    AU --> BJ[Приложение недоступно извне Kubernetes?]:::decision
    BJ -->|ДА| BK[Проблема скорее всего в настройках маршрутизации или внешней доступности]:::step
    BJ -->|НЕТ| BL[Приложение корректно доступно]:::step

    %% Для наглядности связки
    F --> AJ
    H --> AJ
    J --> AJ
    L --> AJ
    N --> AJ
    O --> AJ
    S --> AJ

    %% Конец
    BL --> BM[КОНЕЦ]:::end
    BK --> BM[КОНЕЦ]:::end
    BE --> BM[КОНЕЦ]:::end
    BG --> BI
    BI --> BM[КОНЕЦ]:::end
    AM --> BM
```
