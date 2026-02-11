A) Посмотреть, реально ли процесс vLLM рестартится
docker inspect -f '{{.State.Status}} {{.State.Health.Status}} {{.RestartCount}}' vllm_llm


Если RestartCount растёт — он реально падает.

B) Проверить OOM / GPU ошибки на хосте
dmesg -T | egrep -i 'oom|killed process|xid|nvrm' | tail -n 80
nvidia-smi

docker inspect -f '{{json .Config.Healthcheck}}' vllm_llm

curl --noproxy '*' -sS http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model":"llm",
    "messages":[
      {"role":"system","content":"You are a helpful assistant."},
      {"role":"user","content":"Скажи одним предложением, что такое TEI."}
    ],
    "max_tokens":64,
    "temperature":0.2
  }' | head -c 400; echo

-----


  curl --noproxy '*' -sS http://127.0.0.1:8000/v1/models | head


  docker exec -it vllm_llm bash -lc "curl -v --max-time 3 http://127.0.0.1:8000/health"
docker exec -it vllm_llm bash -lc "curl -v --max-time 3 http://127.0.0.1:8000/v1/models"


docker ps --no-trunc | grep vllm_llm
docker inspect -f '{{.Config.Cmd}}' vllm_llm
docker inspect -f '{{.Config.Entrypoint}}' vllm_llm
docker port vllm_llm

-----


1) Проверь, что внутри контейнера вообще есть shell + кто слушает порт

Сделай так (через sh, без curl):

docker exec -it vllm_llm sh -lc 'ss -lntp || netstat -lntp || (cat /proc/net/tcp && echo "---" && cat /proc/net/tcp6)'


Если ss/netstat нет — это нормально, тогда покажи вывод /proc/net/tcp* (там будет строка с портом 1F40 = 8000 в hex).

2) Проверь с хоста (это главное) — без прокси

С хоста, где крутится docker:

curl --noproxy '*' -sv http://127.0.0.1:8000/health
curl --noproxy '*' -sv http://127.0.0.1:8000/v1/models


Важно: /health у vLLM иногда не существует (и тогда healthcheck будет “unhealthy”). Поэтому /v1/models — самый надёжный.

3) Если /v1/models тоже рвёт соединение — смотри логи на момент запроса
docker logs --tail 200 vllm_llm


И прямо после этого в другом окне повтори:

curl --noproxy '*' -sv http://127.0.0.1:8000/v1/models


Если сервер падает при запросе — в логах будет traceback/ошибка.

Почти наверняка у тебя ещё одна проблема: healthcheck неправильный

У тебя контейнер running but unhealthy, и healthcheck у вас ходит на:
wget -qO- http://localhost:8000/health || exit 1

Если у vLLM нет /health, он всегда будет unhealthy, даже когда API живой.

Правильнее так:

либо /v1/models

либо /v1/health (если у вашей версии есть)

либо просто TCP check

Пример замены на /v1/models:

healthcheck:
  test: ["CMD-SHELL", "wget -qO- http://localhost:8000/v1/models >/dev/null || exit 1"]
  interval: 20s
  timeout: 5s
  retries: 20

Почему у тебя “Соединение разорвано другой стороной” при curl

Это обычно происходит, когда:

ты попал не в vLLM, а в прокси (у тебя так уже было — HTML “Connection to server failed”), или

vLLM принял соединение и тут же закрыл (часто из-за падения воркера/oom/ошибки модели).

Но по последнему скрину ты уже делаешь --noproxy — значит (1) лечится. Остаётся проверить (2) через docker logs + /v1/models.


