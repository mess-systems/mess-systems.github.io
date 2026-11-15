---
title: "Hermes в песочнице: первый этап закрыт"
date: 2026-11-15T17:00:00+03:00
weight: 13
description: "Grok Bot сильно урезает мне лимиты, поэтому я начал делать своего бота. Первый этап пилота: Hermes Agent уже работает в песочнице в k3s — gVisor, закрытый egress, секреты только свои."
---

**Лид:** Grok Bot теперь сильно урезает мне лимиты. Я уже [писал об этом](/ru/blog/ai-pricing/). Значит, пришло время делать своего бота. Сегодня закрыл первый из девяти этапов пилота: приёмка — девять из девяти. Замена — [Hermes Agent](https://hermes-agent.nousresearch.com/) от [Nous Research](https://nousresearch.com/), и он уже работает в песочнице в кластере [k3s](https://k3s.io/). У него своё userspace-ядро, исходящий трафик по умолчанию закрыт, чужие секреты не отдаются. Со стороны это обычная сборка. Пока не начнёшь её собирать.

## Что получилось

Агент живёт в кластере как ресурс Sandbox — это CRD проекта [Agent Sandbox](https://agent-sandbox.sigs.k8s.io/), как раз для одиночных stateful-нагрузок вроде агента. Изоляцию даёт не сам оркестратор, а [RuntimeClass](https://kubernetes.io/docs/concepts/containers/runtime-class/) с [gVisor](https://gvisor.dev/): внутри пода ядро `4.19.0-gvisor`, не хостовое. Хранилище на PVC переживает пересоздание пода.

Сеть — [Cilium](https://cilium.io/). Исходящий трафик по умолчанию закрыт. В [Telegram](https://core.telegram.org/bots/api) агент ходит только через выделенный аутентифицированный прокси [Squid](https://www.squid-cache.org/), и в списке разрешённых адресов ровно один: `api.telegram.org:443`. Прямой выход и любые другие хосты через прокси блокируются. Сообщения агент забирает long polling'ом: сам устанавливает исходящие соединения, снаружи к нему не подключиться.

Секреты приходят из [OpenBao](https://openbao.org/) через [External Secrets Operator](https://external-secrets.io/latest/). Негативный тест — запрос чужого секрета — корректно отклоняется. [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) и [Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/docs/) — без нарушений.

LLM-трафик на этом этапе закрыт. До шлюза агент к моделям не ходит.

План собирали в Cursor. Ansible и kubectl применял сам. Cursor получал SSH только разовыми исключениями на чтение — отладить прокси, когда что-то не сходилось.

## Грабли дня

Без них рассказ был бы нечестным.

- В k3s с [containerd](https://containerd.io/) 2.x плагин CRI сменил имя на `io.containerd.cri.v1.runtime`. Старый конфиг кластер молча игнорировал: вроде всё прописали, а runtime как не было.
- Для gVisor нужен полный архив, с бинарником [Sentry](https://gvisor.dev/docs/) внутри. Урезанного набора не хватает.
- [CiliumClusterwideNetworkPolicy](https://docs.cilium.io/en/stable/network/kubernetes/policy/) с `egressDeny` неявно включила default-deny сразу для всех namespace. External Secrets Operator и контроллер песочниц потеряли доступ к apiserver.
- Squid без `squid -z` не стартует. А директива `never_direct allow all` без вышестоящего прокси отвечает 500 на CONNECT.
- Образ curl, в котором `USER` задан именем, а не числом, ломает `runAsNonRoot`: kubelet не может подтвердить, что процесс не root.

Каждая грабля по отдельности известная. Складываются они только когда собираешь всё вместе.

## Что дальше

Следующий этап — [agentgateway](https://agentgateway.dev/) 1.7, через него пойдёт LLM-трафик. Потом остальные этапы пилота, а выкатку переведём на [Argo CD](https://argo-cd.readthedocs.io/en/stable/).

Мне правда интересно, как это устроено у вас. Как вы изолируете своих агентов: отдельный runtime, сеть, секреты, прокси на выход? Что из этого уже обязательно, а что пока на честном слове? Напишите, сравним подходы.
