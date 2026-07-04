# AI Powered Ghidra

## Premise
I like doing reverse engineering challenges on sites like HackTheBox or when participating in CTF's. One of the things that always takes up most of time though is of course is of course understanding the logic of the functions.

Ghidra does a decent enough job at this level with the codebrowser window, but tracking function calls takes time and sometimes I don't rename which makes me go back and have to rethink what that function is doing.

So as a learning I project, I want to see if I can use a local LLM model to power through this process. I'll be honest, it would probably be overkill for most easy challenges, but I want to explore some things I haven't touched yet, like SOAR like runbooks and potentially leading into using this process for static malware analysis.

## Initial Setup
The list of things for the initial idea:
1. Local LLM model like DeepSeek, I'm trying to stay cheap so it needs to be local, but I'm still looking at which ones would work well for this pipeline. I plan on using Ollama for this.
2. A VM, specifically for doing reverse engineering, primarily with Ghidra. I think I'll just use a basic debian with XFCE here, no reason to use parrot or kali here.
3. Someway to automate aspects of this. I'm currently looking at Shuffle, Open WebUI (this looks more like general conversation interaction.) and maybe some mcp utilization, only because Shuffle is the lead here at the moment.
4. Samples. I'll stick with retired HTB challenges, as I can utilize walkthroughs and control difficulty so I know what's going on and can verify issues/hallucinations.

### Some Initial Notes
I'm also going to explore AI plugins for Ghidra, I see a couple do exist, and I want to see what those do.
