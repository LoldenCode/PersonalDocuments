# Configuring a Self Hosted LLM (Large Language Model) and interacting with it through Rewst
## Content Index

 - [Configuring a Self Hosted LLM (Large Language Model) and interacting with it through Rewst](#Configuring%20a%20Self%20Hosted%20LLM%20(Large%20Language%20Model)%20and%20interacting%20with%20it%20through%20Rewst)
	 - [Installing your LLM](##Installing%20your%20LLM)
		 - [Option 1 - WSL](###Option%201%20-%20WSL)
 - [ShellScript](#!/bin/bash)
	 *  [Option 2 - Windows Native](###Option%202%20-%20Windows%20Native)
	 - [Allow access from Rewst](##Allow%20access%20from%20Rewst)
		 - [Create a component to utilize this](###Create%20a%20component%20to%20utilize%20this)
		 - [Bonus round!](###Bonus%20round!)


Often times, there is sensitive data that you may want to run through a tool like ChatGPT. However, keeping security in mind, this is often not the safest idea. Instead, you can set up a locally hosted LLM that does not have the opportunity to leak any information to the outside world.

<sub>This tutorial is for a deployment in Windows, with an NVIDIA RTX graphics card. If you need assistance with another operating system, or would prefer to use Windows Subsystems for Linux, ask @lolden in Discord, or take a look at [the main GitHub for this project](https://github.com/oobabooga/text-generation-webui)</sub>

## Installing your LLM

We will be using [Ooobabooga](https://github.com/oobabooga/text-generation-webui) for this tutorial. This is a constantly developing project, so some instructions may change over time, but if you can follow along with Option 1, any changes needed should be fairly straightforward from terminal output.

### Option 1 - WSL


1. Configure Windows Subsystem for Linux on your "Server" machine. Oobabooga has basic instructions [here](https://github.com/oobabooga/text-generation-webui/blob/main/docs/WSL-installation-guide.md) for setting it up, along with a v4tov4 portproxy for access over LAN.
2. Clone the latest version of Oobabooga and install it. At the time of writing, October 12, 2023, this is v1.7.
```powershell
	$installPath = "C:\Oobabooga"
	if (!(Test-Path $installPath)) {
		New-Item -ItemType Directory -Force -Path "C:\Oobabooga"
	}
	git clone "https://github.com/oobabooga/text-generation-webui.git" "C:\Oobabooga"
	Set-Location "C:\Oobabooga\text-generation-webui"
	. ./start_wsl.bat
```
3. Follow the installation instructions as they come through. These steps are for an "optimal" "server", so we're assuming an NVIDIA RTX 2060 with 12GB VRAM. If your GPU has less VRAM, some options may change.
4. Once installed, you can run the `./start_wsl.bat` again to launch the local instance.
	- If API or a listening port is desired, modify the `CMD_FLAGS.txt` document by using the `--listen` or `--api` flags.
5. Once logged into the WebUI, navigate to the Model tab at the top. On the right, input the model name you'll be using. For this demonstration, we'll download `TheBloke/dolphin-2.1-mistral-7B-AWQ`. This model should take around 8GB VRAM to load into memory, so again, let me know if you need a smaller model.
6. Once downloaded, in the **Model** dropdown, choose your new model. Click on **Save settings for this model** and then **Reload the Model**.
7. Extract ChatML.yaml to the `.\characters\` subdirectory, and ChatML_inst.yaml to the `.\instruction-templates\` subdirectory.
8. Build out a `.sh` script to kick off your bot - mine looks like such.

```sh
#!/bin/bash
conda activate textgen
python server.py --auto-devices --verbose --model TheBloke_dolphin-2.1-mistral-7B-AWQ --listen --listen-port=xxxx --gpu-memory 9 --character ChatML --api --ssl-keyfile "/home/hahahahahahahaha/certs/key.pem" --ssl-certfile "/home/hahahahahahahaha/certs/cert.pem" --extensions openai api Autobooga code_syntax_highlight Training_PRO
```
9. Hope it works from your WSL terminal. If not then NVIDIA or PyTorch broke something again.

### Option 2 - Windows Native

While not preferred, we can install natively on Windows. Note that this may cause issues with any Python packages you have installed on the system that will be hosting this, and it will run VERY slowly.

1. Download the lastest [All-In-One Windows installer](https://github.com/oobabooga/text-generation-webui/releases/download/installers/oobabooga_windows.zip).
2. Extract this to a folder where it's going to live.
3. Run the start_windows.bat and let it install. Choose your GPU type from the list, or CPU if you do not have a GPU capable of generating text.
4. Run start_windows.bat again once finished.
5. Todo: Finish writing Windows install instructions, need more disk space.

## Allow access from Rewst

Forward port 5000 from your WAN to your new server. Restrict it via firewall to only allow Rewst (`3.139.170.31`)

### Create a component to utilize this

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

Replace that with the following -

  ```json
  {
    "seed":-1,
    "top_a":0,
    "top_k":20,
    "top_p":0.9,
    "preset":"Divine Intellect",
    "prompt":"<insert prompt from HuggingFace>",
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

### Bonus round!

If this is all a bit over your head, and you've made it this far, here is the template file that I use for mine. Note that the URL is removed so you will have to fill that in yourself, and the prompt can be tuned to better fit your use case.
```json
 {
  "e78e2881434d4ee49359d8c03471fcf9": {
    "input": {
      "url": "URL_of_server",
      "body": null,
      "json": {
        "seed": -1,
        "top_a": 0,
        "top_k": 20,
        "top_p": 0.9,
        "preset": "None",
        "prompt": "### Human: {{ CTX.Question }}\r\n### Assistant: ",
        "num_beams": 1,
        "typical_p": 1,
        "min_length": 0,
        "temperature": 0.5,
        "mirostat_eta": 0.1,
        "mirostat_tau": 5,
        "mirostat_mode": 0,
        "penalty_alpha": 0,
        "epsilon_cutoff": 0,
        "length_penalty": 1,
        "max_new_tokens": 800,
        "truncation_length": 2048,
        "repetition_penalty": 1.15,
        "no_repeat_ngram_size": 0,
        "repetition_penalty_range": 0
      },
      "files": [],
      "method": "POST",
      "params": null,
      "cookies": null,
      "headers": null,
      "timeout": 30,
      "auth_password": null,
      "auth_username": null,
      "allow_redirects": true,
      "require_2xx_status": null
    },
    "isMocked": false,
    "mockInput": {
      "mock_result": {}
    },
    "join": 0,
    "metadata": {
      "x": 336,
      "y": 432
    },
    "humanSecondsSaved": 0,
    "transitionMode": "FOLLOW_ALL",
    "packOverrides": [],
    "publishResultAs": "",
    "runAsOrgId": "",
    "with": null,
    "next": [
      {
        "do": [
          "44d7df5121ee41eebe08970a21294d26"
        ],
        "when": "{{ SUCCEEDED }}",
        "label": "",
        "publish": [
          {
            "key": "answer",
            "value": "{{ TASKS.ask_the_bot.result.result.data.results[0].text }}"
          }
        ],
        "id": "b24ea351-258b-4bf2-84ad-40aec6ae0af2",
        "__typename": "WorkflowTransition"
      }
    ],
    "name": "ask_the_bot",
    "description": "Sends CTX.Question to the bot.",
    "timeout": 30,
    "securitySchema": {
      "redact": {
        "input": [
          "auth_password"
        ],
        "result": []
      }
    }
  },
  "44d7df5121ee41eebe08970a21294d26": {
    "name": "core_noop",
    "description": "Action that does nothing",
    "metadata": {
      "x": 336,
      "y": 624
    },
    "next": [
      {
        "do": [],
        "when": "{{ SUCCEEDED }}",
        "label": "",
        "publish": [],
        "id": "0872a250-9a23-4c06-a606-05cd139c3137",
        "__typename": "WorkflowTransition"
      }
    ],
    "input": {},
    "with": null,
    "humanSecondsSaved": 0,
    "join": 0,
    "isMocked": true,
    "mockInput": {
      "mock_result": {
        "bot_says": "{{ CTX.answer }}"
      }
    },
    "timeout": 600,
    "transitionMode": "FOLLOW_ALL",
    "publishResultAs": "",
    "runAsOrgId": "",
    "securitySchema": {
      "redact": {
        "input": [],
        "result": []
      }
    },
    "packOverrides": []
  }
}
```
