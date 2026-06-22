# headroom

Proxy local de otimização de tokens para LLMs (https://github.com/chopratejas/headroom), rodando via Docker em `http://localhost:8787`.

## Subindo o proxy

```bash
docker compose up -d
```

Validar:

```bash
curl http://127.0.0.1:8787/readyz
```

Dashboard web: http://127.0.0.1:8787/dashboard
Métricas (Prometheus): http://127.0.0.1:8787/metrics

Estado do proxy (savings, memória, cache de licença, telemetria) é persistido no volume Docker `headroom-data`, então sobrevive a `docker compose down`/atualizações de imagem.

## Configuração de ambiente

Copie `.env.example` para `.env`. Nenhuma API key é necessária para uso com assinatura (Claude Pro / ChatGPT Plus) — a autenticação é feita via OAuth pelas próprias CLIs/extensões, que o headroom repassa adiante.

## Usando com Claude Code (Claude Pro, extensão VSCode)

Edite `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:8787"
  }
}
```

Isso vale globalmente para CLI e extensão VSCode. Reinicie o VSCode para aplicar. O login OAuth da assinatura continua funcionando — o headroom só intercepta e otimiza o tráfego antes de repassar para `api.anthropic.com`.

## Usando com Codex (ChatGPT Plus, extensão VSCode)

Edite `~/.codex/config.toml` (precisa ser a config de usuário, não a do projeto — chaves como `model_provider`/`model_providers` são ignoradas em `.codex/config.toml` local):

```toml
model_provider = "headroom"

[model_providers.headroom]
name = "Codex via Headroom proxy"
base_url = "http://localhost:8787/v1"
wire_api = "responses"
```

Sem `env_key`: o Codex cai de volta no OAuth bearer + `ChatGPT-Account-ID` da assinatura em vez de exigir API key (suporte adicionado no headroom a partir da v0.24.0; ver [issue #773](https://github.com/chopratejas/headroom/issues/773)).

Reinicie o VSCode para aplicar.

## Usando a CLI do headroom

O serviço `headroom-cli` roda comandos pontuais da CLI (ex.: inspecionar config, ver savings) contra o mesmo estado (`headroom-data`) usado pelo proxy, e está num profile separado (`cli`) para não subir junto com `docker compose up -d`. Por padrão ele lê a config OAuth do Claude/Codex/Gemini do seu `$HOME`; defina `HEADROOM_HOST_HOME` no `.env` apenas se precisar apontar para outro diretório:

```bash
docker compose --profile cli run --rm headroom-cli <comando>
```

Por padrão (sem `<comando>`) ele mostra o `--help`. O diretório atual é montado em `/workspace` (configurável via `HEADROOM_WORKSPACE`).
