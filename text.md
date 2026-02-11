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

