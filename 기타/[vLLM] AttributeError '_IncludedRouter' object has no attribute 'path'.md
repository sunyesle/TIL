# [vLLM] AttributeError: '_IncludedRouter' object has no attribute 'path'

`/v1/models`, `/v1/chat/completions` 등 모든 REST API 엔드포인트에서 `HTTP 500 Internal Server Error`가 발생한다.
```log
(APIServer pid=1) INFO:     Started server process [1]
(APIServer pid=1) INFO:     Waiting for application startup.
(APIServer pid=1) INFO:     Application startup complete.
(APIServer pid=1) INFO:     172.17.0.1:37222 - "GET /v1/models HTTP/1.1" 500 Internal Server Error
(APIServer pid=1) ERROR:    Exception in ASGI application
(APIServer pid=1) Traceback (most recent call last):
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/uvicorn/protocols/http/httptools_impl.py", line 421, in run_asgi
(APIServer pid=1)     result = await app(  # type: ignore[func-returns-value]
(APIServer pid=1)              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/uvicorn/middleware/proxy_headers.py", line 62, in __call__
(APIServer pid=1)     return await self.app(scope, receive, send)
(APIServer pid=1)            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/fastapi/applications.py", line 1162, in __call__
(APIServer pid=1)     await super().__call__(scope, receive, send)
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/starlette/applications.py", line 90, in __call__
(APIServer pid=1)     await self.middleware_stack(scope, receive, send)
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/starlette/middleware/errors.py", line 186, in __call__
(APIServer pid=1)     raise exc
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/starlette/middleware/errors.py", line 164, in __call__
(APIServer pid=1)     await self.app(scope, receive, _send)
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/starlette/middleware/cors.py", line 88, in __call__
(APIServer pid=1)     await self.app(scope, receive, send)
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/prometheus_fastapi_instrumentator/middleware.py", line 131, in __call__
(APIServer pid=1)     handler, is_templated = self._get_handler(request)
(APIServer pid=1)                             ^^^^^^^^^^^^^^^^^^^^^^^^^^
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/prometheus_fastapi_instrumentator/middleware.py", line 240, in _get_handler
(APIServer pid=1)     route_name = routing.get_route_name(request)
(APIServer pid=1)                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/prometheus_fastapi_instrumentator/routing.py", line 75, in get_route_name
(APIServer pid=1)     route_name = _get_route_name(scope, routes)
(APIServer pid=1)                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
(APIServer pid=1)   File "/usr/local/lib/python3.12/dist-packages/prometheus_fastapi_instrumentator/routing.py", line 55, in _get_route_name
(APIServer pid=1)     route_name = route.path
(APIServer pid=1)                  ^^^^^^^^^^
(APIServer pid=1) AttributeError: '_IncludedRouter' object has no attribute 'path'
```

## 원인 및 해결 방법
FastAPI (0.137 이상) 업데이트로 인해 서브 라우터를 다루는 방식이 변경되면서 발생한 vLLM 생태계 공통 버그로, 임시 해결책은 FastAPI를 다운그레이드하는 것이다.

```bash
# 컨테이너 bash에 접속
docker exec -it <컨테이너명> bash

# FastAPI 다운그레이드
pip install "fastapi<0.137"
exit

# 컨테이너 재시작
docker restart <컨테이너명>
```

---
**Reference**
- https://forums.developer.nvidia.com/t/note-latest-updates-break-vllm-see-thread/373314/5
- https://github.com/xorbitsai/inference/issues/5063
- https://github.com/vllm-project/vllm/issues/45596
