# Free-Model Aggregator, Live

An entry for make.fun's **free-model-aggregator-02** ("The best live free-model
aggregator"). One Python 3.13 server, standard library only, that serves an
OpenAI-compatible `POST /v1/chat/completions` and an Anthropic-compatible
`POST /v1/messages` — in JSON and in streaming — and answers every request with
a real completion from a free OpenRouter model, dialled through the trial's
gateway on the entrant's pushed key.

## How it plays

- **Routing (quality):** candidates that meet a prompt's requirements (tools,
  vision, JSON mode, context) are ranked by the brief's published
  `rating_milli`, best first, and the best is dialled first.
- **Failover:** a gateway 429 or 5xx is not the prompt's end. The next
  candidate is dialled (up to 8 calls per prompt), and every refusal along the
  way is reported on the served answer in the `x-makefun-upstream-failures`
  header. A model that refused is demoted behind the models that have not, for
  the rest of the trial, so one rate-limited model cannot sink later prompts.
  All choices derive from the brief, the requests and the gateway's answers —
  no wall clock, no unseeded randomness — so a replayed trial chooses the same.
- **Fidelity:** the answer is returned in the wire the client used (an
  Anthropic-shaped request is translated to Chat Completions and the answer
  translated back), with usage and stop reason whole, and the `model` field
  always names the dialable id of the model that really served the prompt.
  Every dialled body carries an output cap inside the gateway's limit.
- **Catalog:** `--list` prints the catalog document — one `openrouter`
  provider, exactly the brief's fourteen free ids — and the register body
  carries that same document byte for byte. `GET /v1/models` answers in the
  shape of the caller's auth header.

## Run it

Inside a trial the harness sets `MAKEFUN_ENV_URL`, `MAKEFUN_PROMPT_FILE`,
`OPENAI_BASE_URL` and the trial's dummy keys; the server registers its port
and catalog with the env, serves the prompts, polls `GET /work` and exits 0
when the env says done.

```sh
python3 main.py --list   # print the catalog and exit
python3 main.py          # serve a trial
```

## The key

The entrant's OpenRouter key never enters this repository or the sandbox. It
is pushed to the competition itself (`makefun secrets push
free-model-aggregator-02 --env-file .env` with `OPENROUTER_API_KEY` in the
`.env`), and the gateway swaps the trial's dummy key for it on every call.
