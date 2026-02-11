A) Посмотреть, реально ли процесс vLLM рестартится
docker inspect -f '{{.State.Status}} {{.State.Health.Status}} {{.RestartCount}}' vllm_llm


Если RestartCount растёт — он реально падает.

B) Проверить OOM / GPU ошибки на хосте
dmesg -T | egrep -i 'oom|killed process|xid|nvrm' | tail -n 80
nvidia-smi

docker inspect -f '{{json .Config.Healthcheck}}' vllm_llm

