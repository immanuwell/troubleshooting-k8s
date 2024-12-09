# troubleshooting-k8s
Source - learnk8s.io

```mermaid
flowchart TD
    Start(["НАЧАЛО"])
    %% Проверка состояния pod'ов
    Start -->|"kubectl get pods"| A[Есть ли pod'ы в PENDING?]
    A -->|ДА| B[Упирается в лимиты ResourceQuota?]
    B -->|ДА| C(Разрешите больше лимитов кластера)
    B -->|НЕТ| D[Монтируется PersistentVolume, а статус PENDING?]
    D -->|ДА| E(Исправьте лейблы ResourceQuota)
    D -->|НЕТ| F[Копируйте PersistentVolumeClaim]
    F -->|"kubectl describe pod <pod-name>"| G[Pod был назначен узлу?]
    G -->|НЕТ| H(Проблема на стороне Kubelet)
    G -->|ДА| I(Возможно проблема с PersistentVolume или конфигурацией PersistentVolumeClaim)
    A -->|НЕТ| J[Есть ли pod'ы в RUNNING?]
    J -->|ДА| K[Доступны логи приложения?]
    K -->|ДА| L(Исправьте проблему в приложении)
    K -->|НЕТ| M[Контейнер завершился слишком быстро?]
    M -->|ДА| N["kubectl logs pod-name --previous"]
    N -->|"kubectl describe pod pod-name"| O[У образа корректное название?]
    O -->|НЕТ| P(Исправьте название образа)
    O -->|ДА| Q[Статус pod'а = ImagePullBackOff?]
    Q -->|ДА| R(Проблема с образом, исправьте Dockerfile)
    Q -->|НЕТ| S(Обратитесь к StackOverflow)
    M -->|НЕТ| T[Статус pod'а = CrashLoopBackOff?]
    T -->|ДА| U(Проблема с лейблами или переменными окружения)
    T -->|НЕТ| V(Исправьте liveness-probe)
    %% Проверка pod'ов в состоянии READY
    J -->|НЕТ| W[Есть ли pod'ы в READY?]
    W -->|НЕТ| X(Попробуйте определить причину ошибки)
    W -->|ДА| Z[Проблема с readiness-probe?]
    Z -->|ДА| AA(Исправьте readiness-probe)
    Z -->|НЕТ| AB(Проблема с CRI или Kubelet)
    %% Проверка доступа к приложению
    AB -->|"kubectl port-forward pod-name 8080:<pod-port>"| AC[Есть доступ к приложению?]
    AC -->|ДА| AD(Pod'ы корректно функционируют)
    AC -->|НЕТ| AE[Контейнер слушает порт 0.0.0.0?]
    AE -->|НЕТ| AF(Исправьте слушатель, добавьте 0.0.0.0 в описании контейнера)
    AE -->|ДА| AD
    %% Проверка Ingress
    AD -->|"kubectl describe ingress <ingress-name>"| AG[Виден ли список Backends?]
    AG -->|ДА| AH[Значения serviceName и servicePort соответствуют Service?]
    AH -->|НЕТ| AI(Исправьте serviceName и servicePort в Ingress)
    AH -->|ДА| AJ[Получается зайти на приложение?]
    AJ -->|ДА| AK(Ingress корректно функционирует)
    AJ -->|НЕТ| AL(Проблема в конфигурации Ingress или документации)
    %% Проверка Service
    AK -->|"kubectl describe service <service-name>"| AM[Виден список endpoint'ов?]
    AM -->|ДА| AN[Все endpoint'ы принадлежат pod'ам?]
    AN -->|НЕТ| AO(Проблема с Controller Manager)
    AN -->|ДА| AP[Pod'у назначен IP-адрес?]
    AP -->|НЕТ| AQ(Проблема с Controller Manager)
    AP -->|ДА| AR(Проблема в kube-proxy)
    AM -->|НЕТ| AS[Используйте selector в Service, чтобы направлять трафик на pod'ы]
    AS -->|"kubectl port-forward service-name 8080:<service-port>"| AT[Получается зайти на приложение?]
    AT -->|НЕТ| AU(Исправьте targetPort в контейнере и servicePort в Service)
    AT -->|ДА| AK
    %% Завершение
    AL -->|"Приложение должно работать нормально"| End(["КОНЕЦ"])
    classDef blueFill fill:#d2e7f7,stroke:#000000,stroke-width:1px;
    class C,E,H,I,L,P,R,S,U,V,X,Y,AA,AB,AD,AF,AI,AK,AL,AO,AQ,AR,AU blueFill;
```

![](https://mermaid.ink/img/eyJjb2RlIjoiXG5mbG93Y2hhcnQgVERcbiAgICBTdGFydChbXCLQndCQ0KfQkNCb0J5cIl0pXG4gICAgJSUg0J_RgNC-0LLQtdGA0LrQsCDRgdC-0YHRgtC-0Y_QvdC40Y8gcG9kJ9C-0LJcbiAgICBTdGFydCAtLT58XCJrdWJlY3RsIGdldCBwb2RzXCJ8IEFb0JXRgdGC0Ywg0LvQuCBwb2Qn0Ysg0LIgUEVORElORz9dXG4gICAgQSAtLT580JTQkHwgQlvQo9C_0LjRgNCw0LXRgtGB0Y8g0LIg0LvQuNC80LjRgtGLIFJlc291cmNlUXVvdGE_XVxuICAgIEIgLS0-fNCU0JB8IEMo0KDQsNC30YDQtdGI0LjRgtC1INCx0L7Qu9GM0YjQtSDQu9C40LzQuNGC0L7QsiDQutC70LDRgdGC0LXRgNCwKVxuICAgIEIgLS0-fNCd0JXQonwgRFvQnNC-0L3RgtC40YDRg9C10YLRgdGPIFBlcnNpc3RlbnRWb2x1bWUsINCwINGB0YLQsNGC0YPRgSBQRU5ESU5HP11cbiAgICBEIC0tPnzQlNCQfCBFKNCY0YHQv9GA0LDQstGM0YLQtSDQu9C10LnQsdC70YsgUmVzb3VyY2VRdW90YSlcbiAgICBEIC0tPnzQndCV0KJ8IEZb0JrQvtC_0LjRgNGD0LnRgtC1IFBlcnNpc3RlbnRWb2x1bWVDbGFpbV1cbiAgICBGIC0tPnxcImt1YmVjdGwgZGVzY3JpYmUgcG9kIDxwb2QtbmFtZT5cInwgR1tQb2Qg0LHRi9C7INC90LDQt9C90LDRh9C10L0g0YPQt9C70YM_XVxuICAgIEcgLS0-fNCd0JXQonwgSCjQn9GA0L7QsdC70LXQvNCwINC90LAg0YHRgtC-0YDQvtC90LUgS3ViZWxldClcbiAgICBHIC0tPnzQlNCQfCBJKNCS0L7Qt9C80L7QttC90L4g0L_RgNC-0LHQu9C10LzQsCDRgSBQZXJzaXN0ZW50Vm9sdW1lINC40LvQuCDQutC-0L3RhNC40LPRg9GA0LDRhtC40LXQuSBQZXJzaXN0ZW50Vm9sdW1lQ2xhaW0pXG4gICAgQSAtLT580J3QldCifCBKW9CV0YHRgtGMINC70LggcG9kJ9GLINCyIFJVTk5JTkc_XVxuICAgIEogLS0-fNCU0JB8IEtb0JTQvtGB0YLRg9C_0L3RiyDQu9C-0LPQuCDQv9GA0LjQu9C-0LbQtdC90LjRjz9dXG4gICAgSyAtLT580JTQkHwgTCjQmNGB0L_RgNCw0LLRjNGC0LUg0L_RgNC-0LHQu9C10LzRgyDQsiDQv9GA0LjQu9C-0LbQtdC90LjQuClcbiAgICBLIC0tPnzQndCV0KJ8IE1b0JrQvtC90YLQtdC50L3QtdGAINC30LDQstC10YDRiNC40LvRgdGPINGB0LvQuNGI0LrQvtC8INCx0YvRgdGC0YDQvj9dXG4gICAgTSAtLT580JTQkHwgTltcImt1YmVjdGwgbG9ncyBwb2QtbmFtZSAtLXByZXZpb3VzXCJdXG4gICAgTiAtLT58XCJrdWJlY3RsIGRlc2NyaWJlIHBvZCBwb2QtbmFtZVwifCBPW9CjINC-0LHRgNCw0LfQsCDQutC-0YDRgNC10LrRgtC90L7QtSDQvdCw0LfQstCw0L3QuNC1P11cbiAgICBPIC0tPnzQndCV0KJ8IFAo0JjRgdC_0YDQsNCy0YzRgtC1INC90LDQt9Cy0LDQvdC40LUg0L7QsdGA0LDQt9CwKVxuICAgIE8gLS0-fNCU0JB8IFFb0KHRgtCw0YLRg9GBIHBvZCfQsCA9IEltYWdlUHVsbEJhY2tPZmY_XVxuICAgIFEgLS0-fNCU0JB8IFIo0J_RgNC-0LHQu9C10LzQsCDRgSDQvtCx0YDQsNC30L7QvCwg0LjRgdC_0YDQsNCy0YzRgtC1IERvY2tlcmZpbGUpXG4gICAgUSAtLT580J3QldCifCBTKNCe0LHRgNCw0YLQuNGC0LXRgdGMINC6IFN0YWNrT3ZlcmZsb3cpXG4gICAgTSAtLT580J3QldCifCBUW9Ch0YLQsNGC0YPRgSBwb2Qn0LAgPSBDcmFzaExvb3BCYWNrT2ZmP11cbiAgICBUIC0tPnzQlNCQfCBVKNCf0YDQvtCx0LvQtdC80LAg0YEg0LvQtdC50LHQu9Cw0LzQuCDQuNC70Lgg0L_QtdGA0LXQvNC10L3QvdGL0LzQuCDQvtC60YDRg9C20LXQvdC40Y8pXG4gICAgVCAtLT580J3QldCifCBWKNCY0YHQv9GA0LDQstGM0YLQtSBsaXZlbmVzcy1wcm9iZSlcbiAgICAlJSDQn9GA0L7QstC10YDQutCwIHBvZCfQvtCyINCyINGB0L7RgdGC0L7Rj9C90LjQuCBSRUFEWVxuICAgIEogLS0-fNCd0JXQonwgV1vQldGB0YLRjCDQu9C4IHBvZCfRiyDQsiBSRUFEWT9dXG4gICAgVyAtLT580J3QldCifCBYKNCf0L7Qv9GA0L7QsdGD0LnRgtC1INC-0L_RgNC10LTQtdC70LjRgtGMINC_0YDQuNGH0LjQvdGDINC-0YjQuNCx0LrQuClcbiAgICBXIC0tPnzQlNCQfCBaW9Cf0YDQvtCx0LvQtdC80LAg0YEgcmVhZGluZXNzLXByb2JlP11cbiAgICBaIC0tPnzQlNCQfCBBQSjQmNGB0L_RgNCw0LLRjNGC0LUgcmVhZGluZXNzLXByb2JlKVxuICAgIFogLS0-fNCd0JXQonwgQUIo0J_RgNC-0LHQu9C10LzQsCDRgSBDUkkg0LjQu9C4IEt1YmVsZXQpXG4gICAgJSUg0J_RgNC-0LLQtdGA0LrQsCDQtNC-0YHRgtGD0L_QsCDQuiDQv9GA0LjQu9C-0LbQtdC90LjRjlxuICAgIEFCIC0tPnxcImt1YmVjdGwgcG9ydC1mb3J3YXJkIHBvZC1uYW1lIDgwODA6PHBvZC1wb3J0PlwifCBBQ1vQldGB0YLRjCDQtNC-0YHRgtGD0L8g0Log0L_RgNC40LvQvtC20LXQvdC40Y4_XVxuICAgIEFDIC0tPnzQlNCQfCBBRChQb2Qn0Ysg0LrQvtGA0YDQtdC60YLQvdC-INGE0YPQvdC60YbQuNC-0L3QuNGA0YPRjtGCKVxuICAgIEFDIC0tPnzQndCV0KJ8IEFFW9Ca0L7QvdGC0LXQudC90LXRgCDRgdC70YPRiNCw0LXRgiDQv9C-0YDRgiAwLjAuMC4wP11cbiAgICBBRSAtLT580J3QldCifCBBRijQmNGB0L_RgNCw0LLRjNGC0LUg0YHQu9GD0YjQsNGC0LXQu9GMLCDQtNC-0LHQsNCy0YzRgtC1IDAuMC4wLjAg0LIg0L7Qv9C40YHQsNC90LjQuCDQutC-0L3RgtC10LnQvdC10YDQsClcbiAgICBBRSAtLT580JTQkHwgQURcbiAgICAlJSDQn9GA0L7QstC10YDQutCwIEluZ3Jlc3NcbiAgICBBRCAtLT58XCJrdWJlY3RsIGRlc2NyaWJlIGluZ3Jlc3MgPGluZ3Jlc3MtbmFtZT5cInwgQUdb0JLQuNC00LXQvSDQu9C4INGB0L_QuNGB0L7QuiBCYWNrZW5kcz9dXG4gICAgQUcgLS0-fNCU0JB8IEFIW9CX0L3QsNGH0LXQvdC40Y8gc2VydmljZU5hbWUg0Lggc2VydmljZVBvcnQg0YHQvtC-0YLQstC10YLRgdGC0LLRg9GO0YIgU2VydmljZT9dXG4gICAgQUggLS0-fNCd0JXQonwgQUko0JjRgdC_0YDQsNCy0YzRgtC1IHNlcnZpY2VOYW1lINC4IHNlcnZpY2VQb3J0INCyIEluZ3Jlc3MpXG4gICAgQUggLS0-fNCU0JB8IEFKW9Cf0L7Qu9GD0YfQsNC10YLRgdGPINC30LDQudGC0Lgg0L3QsCDQv9GA0LjQu9C-0LbQtdC90LjQtT9dXG4gICAgQUogLS0-fNCU0JB8IEFLKEluZ3Jlc3Mg0LrQvtGA0YDQtdC60YLQvdC-INGE0YPQvdC60YbQuNC-0L3QuNGA0YPQtdGCKVxuICAgIEFKIC0tPnzQndCV0KJ8IEFMKNCf0YDQvtCx0LvQtdC80LAg0LIg0LrQvtC90YTQuNCz0YPRgNCw0YbQuNC4IEluZ3Jlc3Mg0LjQu9C4INC00L7QutGD0LzQtdC90YLQsNGG0LjQuClcbiAgICAlJSDQn9GA0L7QstC10YDQutCwIFNlcnZpY2VcbiAgICBBSyAtLT58XCJrdWJlY3RsIGRlc2NyaWJlIHNlcnZpY2UgPHNlcnZpY2UtbmFtZT5cInwgQU1b0JLQuNC00LXQvSDRgdC_0LjRgdC-0LogZW5kcG9pbnQn0L7Qsj9dXG4gICAgQU0gLS0-fNCU0JB8IEFOW9CS0YHQtSBlbmRwb2ludCfRiyDQv9GA0LjQvdCw0LTQu9C10LbQsNGCIHBvZCfQsNC8P11cbiAgICBBTiAtLT580J3QldCifCBBTyjQn9GA0L7QsdC70LXQvNCwINGBIENvbnRyb2xsZXIgTWFuYWdlcilcbiAgICBBTiAtLT580JTQkHwgQVBbUG9kJ9GDINC90LDQt9C90LDRh9C10L0gSVAt0LDQtNGA0LXRgT9dXG4gICAgQVAgLS0-fNCd0JXQonwgQVEo0J_RgNC-0LHQu9C10LzQsCDRgSBDb250cm9sbGVyIE1hbmFnZXIpXG4gICAgQVAgLS0-fNCU0JB8IEFSKNCf0YDQvtCx0LvQtdC80LAg0LIga3ViZS1wcm94eSlcbiAgICBBTSAtLT580J3QldCifCBBU1vQmNGB0L_QvtC70YzQt9GD0LnRgtC1IHNlbGVjdG9yINCyIFNlcnZpY2UsINGH0YLQvtCx0Ysg0L3QsNC_0YDQsNCy0LvRj9GC0Ywg0YLRgNCw0YTQuNC6INC90LAgcG9kJ9GLXVxuICAgIEFTIC0tPnxcImt1YmVjdGwgcG9ydC1mb3J3YXJkIHNlcnZpY2UtbmFtZSA4MDgwOjxzZXJ2aWNlLXBvcnQ-XCJ8IEFUW9Cf0L7Qu9GD0YfQsNC10YLRgdGPINC30LDQudGC0Lgg0L3QsCDQv9GA0LjQu9C-0LbQtdC90LjQtT9dXG4gICAgQVQgLS0-fNCd0JXQonwgQVUo0JjRgdC_0YDQsNCy0YzRgtC1IHRhcmdldFBvcnQg0LIg0LrQvtC90YLQtdC50L3QtdGA0LUg0Lggc2VydmljZVBvcnQg0LIgU2VydmljZSlcbiAgICBBVCAtLT580JTQkHwgQUtcbiAgICAlJSDQl9Cw0LLQtdGA0YjQtdC90LjQtVxuICAgIEFMIC0tPnxcItCf0YDQuNC70L7QttC10L3QuNC1INC00L7Qu9C20L3QviDRgNCw0LHQvtGC0LDRgtGMINC90L7RgNC80LDQu9GM0L3QvlwifCBFbmQoW1wi0JrQntCd0JXQplwiXSlcbiAgICBjbGFzc0RlZiBibHVlRmlsbCBmaWxsOiNkMmU3Zjcsc3Ryb2tlOiMwMDAwMDAsc3Ryb2tlLXdpZHRoOjFweDtcbiAgICBjbGFzcyBDLEUsSCxJLEwsUCxSLFMsVSxWLFgsWSxBQSxBQixBRCxBRixBSSxBSyxBTCxBTyxBUSxBUixBVSBibHVlRmlsbDtcbiIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In19)
