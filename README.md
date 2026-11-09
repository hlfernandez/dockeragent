# This image belongs to a larger project called Bioinformatics Docker Images Project (http://pegi3s.github.io/dockerfiles)
## (Please note that the original software licenses still apply)

This image facilitates the usage of [dockeragent](https://github.com/pegi3s/dockeragent), an AI agent that turns a GitHub repository URL into a working, tested (if suitable data is available) pegi3s-style Bioinformatics Docker image, and that also generates the needed metadata (Dockerfile, README, metadata.json, License, etc.), ready for submission to [pegi3s BDIP](https://bdip.i3s.up.pt/). This Docker image has been used to generate the needed files for software applications with no special instalation requirements, for Jupyter Notebook applications (in this case the code is converted to Python scripts, for NextFlow applications where Docker is a wrapper that facilitates the running of the application, and R applications where a Rscript is generated to avoid having to enter the image and to declare each command separately.

## If you use this agent to generate a Docker image for a software not available at pegi3s BDIP, please consider, after testing it, to contribute the Dockerfile and associated metadata (including the test data) to the pegi3s BDIP project, by sending an e-mail message to pegi3sdocker@gmail.com with the `for_submission/` folder as an attachment. If you generate Docker images for a different purpose, using this Docker image, you must declare that you have used dockeragent to generate them.

## Requirements

Before running the image you need two files on the host:

1. An API key file at `/opt/.apikey` — a plain text file containing **only** your AI provider's API key.
2. A `config.json` file in the directory you mount as `/data` — describes which AI provider/model dockeragent should use.

### `/opt/.apikey`

Create the file `/opt/.apikey` on the host containing a single line with your API key, e.g.:

```
sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

No quotes, no trailing whitespace, no other content. This directory is mounted read-only-in-spirit into the container with `-v /opt:/opt`, so dockeragent can read it at `/opt/.apikey`.

### `config.json`

Place a `config.json` file in the directory you will mount as `/data` (e.g. your working directory). The main providers are:

| Provider    | `npm`                      | `base_url`                                                 |
|-------------|----------------------------|------------------------------------------------------------|
| DeepSeek    | `@ai-sdk/openai-compatible` | `https://api.deepseek.com/v1`                              |
| OpenCode Go | `@ai-sdk/openai-compatible` | `https://opencode.ai/inference/go/openai/v1`               |
| OpenAI      | `@ai-sdk/openai`           | `https://api.openai.com/v1`                                |
| Anthropic   | `@ai-sdk/anthropic`        | `https://api.anthropic.com`                                |
| Gemini      | `@ai-sdk/google`           | `https://generativelanguage.googleapis.com/v1beta`         |

All six keys are required: `provider_id`, `provider_name`, `npm`, `base_url`, `model_id`, `model_name`.

#### DeepSeek example

Example using DeepSeek:

```json
{
  "provider_id":   "deepseek",
  "provider_name": "DeepSeek",
  "npm":           "@ai-sdk/openai-compatible",
  "base_url":      "https://api.deepseek.com/v1",
  "model_id":      "deepseek-flash",
  "model_name":    "DeepSeek V4.1 Flash"
}
```

#### OpenCode Go example

Example using OpenCode Go (requires an OpenCode Go subscription key):

```json
{
  "provider_id":   "opencode-go",
  "provider_name": "OpenCode Go",
  "npm":           "@ai-sdk/openai-compatible",
  "base_url":      "https://opencode.ai/inference/go/openai/v1",
  "model_id":      "deepseek-v4.1-flash",
  "model_name":    "DeepSeek V4.1 Flash"
}
```

Keep `provider_id` set to `opencode-go`: OpenCode uses the `opencode` prefix to add the `x-opencode-session` header required by the OpenCode Go API. The list of available models is published at `https://opencode.ai/inference/go/v1/models`.

# Using the dockeragent image in Linux

You should adapt and run the following command:

```
sudo docker run -v /var/run/docker.sock:/var/run/docker.sock -v /your/data/dir:/data -v /opt:/opt pegi3s/dockeragent run "https://github.com/<user>/<repo>"
```

In this command, you should replace:
- `/your/data/dir` to point to the directory containing your `config.json` (this is also where the results will be written).
- `https://github.com/<user>/<repo>` to the GitHub URL of the tool you want to dockerize.

Example:

```
sudo docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/data -v /opt:/opt pegi3s/dockeragent run "https://github.com/unipept/FragGeneScanRs"
```

Notes on the mounts:
- `/var/run/docker.sock` gives dockeragent access to the host's Docker daemon, which it needs to build and test the generated image.
- `/opt:/opt` gives dockeragent access to the API key at `/opt/.apikey`.
- `/your/data/dir:/data` gives dockeragent access to `config.json` and is where it writes its output.

To run in debug mode (prints the full commands, outputs and reasoning for every step), append `-d` after the GitHub URL:

```
sudo docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/data -v /opt:/opt pegi3s/dockeragent run "https://github.com/putonti/virMine -d"
```

## Output

Results are written to `/your/data/dir/dockeragent_results/<program_name>/`, including the generated `Dockerfile`, `README.md`, `metadata.json`, `ontology.diaf`, `License` and a `for_submission/` folder laid out exactly as needed for submission to pegi3s BDIP.
