# Configuring a Self Hosted LLM (Large Language Model) with it through Rewst

Often times, there is sensitive data that you may want to run through a tool like ChatGPT. However, keeping security in mind, this is often not the safest idea. Instead, you can set up a locally hosted LLM that does not have the opportunity to leak any information to the outside world.

<sub>This tutorial is for a deployment in Windows, with an NVIDIA RTX graphics card. If you need assistance with another operating system, or would prefer to use Windows Subsystems for Linux, ask in the Discord! @lolden (and plenty of others) should be able to help get you on your way!</sub>

## Installing your LLM

We will be using [Ooobabooga](https://github.com/oobabooga/text-generation-webui) for this tutorial. This is a constantly developing project, so while you may be able to operate off the latest build in their GitHub, for the purposes of this tutorial we're going to use [Release v1.3.1](https://github.com/oobabooga/text-generation-webui/releases/tag/v1.3.1)

### Option 1 - Docker

1. Download Choco package manager.

    * ```ps
      Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
        ```

2. Download drivers and dependencies.

```ps
choco install nvidia-display-driver cuda git docker-desktop
```

* At this step, I personally ran into an error during the install of cuda. After attempting to run it manually (`%temp%\chocolatey\cuda\<vers>\cuda_windows.exe`) I found that a system restart was required.

4. Install WSL.

```ps
wsl --install
```

5. Reboot your system.
6. Open a PowerShell session and clone the repo to a location you'd prefer. In this example, we're going to use the `C:\TextGen\` folder.

```ps
cd C:\TextGen
git clone https://github.com/oobabooga/text-generation-webui
```

7. Download and place your models in the model folder.
   1. If your GPU has at least 10GB VRAM, we'll use the [Llama-2-13B-chat-GPTQ](https://huggingface.co/TheBloke/Llama-2-13B-chat-GPTQ) model. Navigate to the root folder (`C:\TextGen\text-generation-webui` in our example) and run `python .\download-model.py TheBloke/Llama-2-13B-chat-GPTQ`.
   2. If you are running a lower end RTX card, we'll use the 7B version. This takes roughly half the VRAM, but is trained on only 7 billion tokens, rather than the 13 billion of the prior. Navigate to the root folder (`C:\TextGen\text-generation-webui` in our example) and run `python .\download-model.py TheBloke/Llama-2-7B-chat-GPTQ`.
8. Copy the docker files to your root folder.

```ps
copy .\docker\* .\
copy .env.example .env
notepad .env
```

9. Modify the newly opened text document, and change the CLI_ARGS to `--model TheBloke_Llama-2-13B-chat-GPTQ --wbits 4 --groupsize 128 --loader exllama_hf --verbose --listen --auto-devices --api` (replace 13B with 7B if followed in step 6)
10. Run `docker compose up`. This will take a few minutes to build the first time.
11. Verify the webUI is working (Navigate to `http://localhost:7860`)
12. If accessible, continue on.

### Option 2 - Windows Native

While not preferred, we can install natively on Windows. Note that this may cause issues with any Python packages you have installed on the system that will be hosting this, and it will run VERY slowly.

1. Download the lastest [All-In-One Windows installer](https://github.com/oobabooga/text-generation-webui/releases/download/installers/oobabooga_windows.zip).
2. Extract this to a folder where it's going to live.
3. Run the start_windows.bat and let it install. Choose your GPU type from the list, or CPU if you do not have a GPU capable of generating text.
4. Run start_windows.bat again once finished.
5. Todo: Finish writing Windows install instructions, need more disk space. 

## Allow access from Rewst

Forward port 5000 from your WAN to your new server. Restrict it via firewall to only allow Rewst (`3.139.170.31`)

## Create a component to utilize this

1. Create a new Workflow.
2. Add the Generic HTTP Request component
3. Configure as follows:
   1. POST
   2. URL: `http://<publicIP>/api/v1/generate`
   3. JSON: create a dummy key and value
   4. Timeout: 60 seconds (May need more, depending on your TextGen model and inference hardware)
4. Edit the raw code of the HTTP request. Where you have your dummy code:

```json
{
  "dummy": "dummy"
}
```

  Replace it with the following:

```json
{
  "seed":-1,
  "top_a":0,
  "top_k":20,
  "top_p":0.9,
  "preset":"Divine Intellect",
  "prompt":"[INST] <<SYS>>\nAnswer the questions.\n<</SYS>>\n{{ CTX.Question }} [/INST]",
  "num_beams":1,
  "typical_p":1,
  "min_length":0,
  "temperature":0.7,
  "mirostat_eta":0.1,
  "mirostat_tau":5,
  "mirostat_mode":0,
  "penalty_alpha":0,
  "epsilon_cutoff":0,
  "length_penalty":1,
  "max_new_tokens":800,
  "truncation_length":2048,
  "repetition_penalty":1.15,
  "no_repeat_ngram_size":0,
  "repetition_penalty_range":0
  }
```

5. Create an input on this workflow that is required. Name it `Question`.
6. Test your new workflow! You should see the output in the workflow results.