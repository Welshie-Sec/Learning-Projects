# GhidrAssist and LM Studio

So for this first section, I'm going to do an initial setup of GhidraAssist, which is a plugin for Ghidra that utilizes API calls to a specified AI model (cloud or local). Now, being unemployed currently, I need to keep things cheap, so I'm going to go with the local route using LM Studio and test some models against HackTheBox retired challenges.

## Remnux for Analysis - First Attempt

I'm grabbing the OVA from the Remnux site and starting from there, as it seems like they've recently included AI tie-ins, so why reinvent the wheel?

* Looks like on first boot up, if I check Ghidra Extensions, GhidrAssistMCP is already checked (...and red?), so this part seems like we're in a good spot.
* Clipboard transfer works right from the get-go as well, so for basic functionality in terms of this project, we seem to be ready to rock.

## First Day Setup

First-day setup wasn't bad. I initially tried a couple of different methods of hosting on my Windows client that I wasn't satisfied with. Initially, the idea was to use Ollama, but I shied away from it when looking through the GhidrAssist recommendations. I did also try to use the Windows Docker app, but that was not a great experience. For example, my C drive isn't a good place for all these to be stored, and I have a 4TB drive I want everything to go on. It boggles my mind that the Docker Windows setup binary does not have a place to make that selection, but does take runtime arguments, so you can specify from the command line. Windows Docker also appears to allocate memory for containers prior to even pulling any containers, so the initial choking on my C drive because of some crazy amount of writes happening and taking all of my client's available RAM while nothing was set up yet really had me looking into alternatives.

LM Studio, based on the GhidrAssist GitHub recommendation, worked really well here. Fast and easy to set up where I want it, and I didn't have to worry about memory juggling until I loaded a model.

### The First Models Tested

So running an LLM at home, I have the basis for what I need; now I just need to pick a model. This is my first time interacting with one in some time (the first time really doesn't count because it was something like diffuse image generation, so whatever). Now let's play spot the misstep.



Right, so based on GhidrAssist references, I wanted to try Qwen out, and I thought, "Right, I'll just ask it to make a convoluted hello world in Python as a test" and see how long it takes to come back. I even left thinking on for the 27B (12GB VRAM) model. Safe to say, after not being able to use my computer for 16 minutes as it decided to keep adding layers of convolutedness, if I'm going to use this for a proper process... I need to use my resources in a better way.

So instead, I went with 3.5 9B, and it's actually pretty quick. And for at least testing the process, I eventually decided this would work.

---

## Back to Remnux: GhidrAssistMCP Setup

Now, for this first pass, I just want the model to tell me things about the target file in the project by running the GhidrAssist commands.

Because Remnux comes with GhidrAssist and GhidrAssistMCP right out of the image, this solves having to initially get the plugin installed.

### Steps for Getting GhidrAssistMCP Going

**1. In Ghidra's initial project view window:**
Navigate to `File -> Install Extensions`. Make sure the GhidrAssistMCP checkbox is selected. In my case, it's red for some reason and I don't know what that identifies (maybe version?).

> **Caution:** The next step requires the CodeBrowser window being open, so I recommend you use a binary you are familiar with.

**2. In CodeBrowser:**
Navigate to `File -> Configure -> Miscellaneous Configure`. Hit the checkmark next to `GhidrAssistMCPPlugin` and after clicking okay, back in the Configure Tool, have the `Miscellaneous` selection box checkmarked as well.
*Note: Without this step, the configuration window we are going to next just won't exist.*

**3. Back in CodeBrowser:**
Navigate to `Window -> GhidrAssistMCP`.

**4. GhidrAssistMCP configuration:**
Here I only made one change, and the change I made was changing the default `localhost` to `0.0.0.0`, because the default only made the port available locally to the VM, whereas I'm running the model on the client. It is not recommended for a production environment to be this basic, and definitely not recommended for any environment with active malware. I will not be teaching you how to harden in that way as I currently do not know at the moment.

**5. Back to LM Studio:**
Hopefully, you weren't waiting until this step to download your model, as it takes forever :^). Go into the Developer app, and at the top, there is an `mcp.json` button. Here we are going to put this configuration in. It's super basic, and as I learn more, I know I'll be modifying it.

```json
{
  "mcpServers": {
    "GhidraAssist": {
      "url": "http://[yourGhidrasIP]:8080/mcp"
    }
  }
}

```

Save it and put the status from stopped to running if you haven't already.

We can then check in LM Studio under `Settings -> Integrations` and see if it sees our new tool.

*[Image]*

---

## Testing the Integration

So to test the usability here, I decided to put it in a tough spot off the start. The file I have open in Ghidra is called "headache," which is a medium retired reversing challenge from HackTheBox, and also one I haven't solved myself yet. Long story short, through the writeup you can see the challenge deals with metamorphic code creation and several anti-debugging layers you have to work through, which is way beyond what I asked the LLM to deal with at the moment. But I started off easy by asking it to write a synopsis of the file and, for bonus points, to give me a potential Sigma rule to look over and judge.

*[Images here]*

*[And here]*

*[And here]*

I find the results interesting, but I'm not completely confident in some aspects:

* The report is very "Beware, malware" about everything. Like, `mmap` is just memory allocation, so only applying it to ROP chains, code injection, yada yada feels a bit too alarmist.
* As expected, it didn't really dig too deep on its own, as it didn't seem to try to untwist and get the full picture of what the program is doing.
* The Sigma rule I'm not well versed in, but it looks like it focused on `ptrace` and just elf binaries in general, which wouldn't be a good addition. I feel like it is more fantasy than a good rule creation. Like it detected native code in `.text`; I feel like that's a much stronger indicator.

I did a couple of rounds to practice the prompt engineering and see if it can actually solve the layers and get the flag, but it is not happy at the moment. As I'm currently watching it search bytes for some patterns, it's been looping like this for a while, and I think I'm going to run out of context tokens pretty fast this way (glad I didn't hook this up to a chargeable API yet).

This is definitely a working base, and I need to explore some more aspects of what it can and can't do. I also need to see if the entire pipeline is up to date, because I'm looking at the GhidrAssist GitHub, and some things are definitely different right off the start. Most notably, I do want to figure out how to have the model rename functions and be more human-readable, like through an initial pass, but at the same time, I'm interested in seeing if it can just solve the challenges as well.