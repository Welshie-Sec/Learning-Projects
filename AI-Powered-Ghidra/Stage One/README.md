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

So to test the usability here, I decided to put it in a tough spot off the start. The file I have open in Ghidra is called "headache," which is a medium retired reversing challenge from HackTheBox, and also one I haven't solved myself yet. Long story short, through the writeup you can see the challenge deals with metamorphic code creation and several anti-debugging layers you have to work through, which is way beyond what I asked the LLM to deal with at the moment. But I started off easy by asking it to write a synopsis of the file and, for bonus points, to give me a potential detection rule to look over and judge.

*[Images here]*

*[And here]*

*[And here]*

I find the results interesting, but I'm not completely confident in some aspects:

* The report is very "Beware, malware" about everything. Like, `mmap` is just memory allocation, so only applying it to ROP chains, code injection, yada yada feels a bit too alarmist.
* As expected, it didn't really dig too deep on its own, as it didn't seem to try to untwist and get the full picture of what the program is doing.
* The rule I'm not well versed in but seems to be a combination of Yara and Sigma, and that was my fault as I told it to make a Sigma rule for something that isn't a log lol, but it looks like it focused on `ptrace` and just elf binaries in general, which wouldn't be a good addition. I feel like it is more fantasy than a good rule creation. Like it detected native code in `.text`; I feel like that's a much stronger indicator.

I did a couple of rounds to practice the prompt engineering and see if it can actually solve the layers and get the flag, but it is not happy at the moment. As I'm currently watching it search bytes for some patterns, it's been looping like this for a while, and I think I'm going to run out of context tokens pretty fast this way (glad I didn't hook this up to a chargeable API yet).

This is definitely a working base, and I need to explore some more aspects of what it can and can't do. I also need to see if the entire pipeline is up to date, because I'm looking at the GhidrAssist GitHub, and some things are definitely different right off the start. Most notably, I do want to figure out how to have the model rename functions and be more human-readable, like through an initial pass, but at the same time, I'm interested in seeing if it can just solve the challenges as well.


# Day 2 - Testing Models

So for the start of day 2 I'm doing a couple of things. First I'm updating Remnux to see if this will update GhidrAssist for some reason as the amount of functionality between the two with available tools is pretty massive. Like version 12 can mostly look at the code but the github shows functions for patching and what not so I want to get there.
To update Remnux in general it uses the command ```remnux install``` which threw me for a bit of a loop, because I was generally just expecting to do things in a package manager. So while I wait for that to update, I'm trying out models on some basic things. I'm using the prompt:

```Make me a convoluted HTML webpage that only shows Hello World.```

And seeing the results, it's pretty open ended, and allows me to see with thinking and reasoning on, how the model generally handle themselves.

### google/gemma-4-12b-qat

11.10 seconds on the first run and made this:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System_Initialize_v4.0.2</title>
    <style>
        :root {
            --bg-color: #0a0a0a;
            --accent-color: #00ff41;
            --font-main: 'Courier New', Courier, monospace;
            --render-speed: 0.5s;
        }

        body {
            background-color: var(--bg-color);
            color: var(--accent-color);
            margin: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            font-family: var(--font-main);
        }

        #viewport {
            position: relative;
            border: 1px solid var(--accent-color);
            padding: 50px;
            box-shadow: 0 0 20px rgba(0, 255, 65, 0.2);
        }

        .glitch-char {
            display: inline-block;
            transition: all var(--render-speed) ease-out;
            opacity: 0;
            transform: translateY(20px) scale(0.5);
        }

        .glitch-char.visible {
            opacity: 1;
            transform: translateY(0) scale(1);
        }

        #status-bar {
            position: absolute;
            bottom: 0;
            left: 0;
            font-size: 10px;
            width: 100%;
            text-align: right;
            text-transform: uppercase;
        }
    </style>
</head>
<body>

    <div id="viewport">
        <div id="render-target"></div>
        <div id="status-bar">Kernel_Status: Initializing...</div>
    </div>

    <script>
        /**
         * CONVOLUTED RENDERING ENGINE
         * Purpose: To display a string that is hidden from the static HTML.
         */
        const Engine = (() => {
            // The "Payload" is Base64 encoded to prevent simple grep discovery
            const _encoded_data = "SGVsbG8sIFdvcmxkIQ=="; 
            const _target = document.getElementById('render-target');
            const _status = document.getElementById('status-bar');

            // Create a Proxy to intercept "rendering" calls
            const renderProxy = new Proxy({}, {
                get: function(target, prop) {
                    if (prop === 'execute') {
                        return () => {
                            const decoded = atob(_encoded_data);
                            initSequence(decoded);
                        };
                    }
                }
            });

            function initSequence(str) {
                _status.innerText = "Kernel_Status: Decrypting Payload...";
                
                // Use a timeout to simulate "processing"
                setTimeout(() => {
                    _status.innerText = "Kernel_Status: Mounting DOM Nodes...";
                    
                    // Create elements dynamically via a recursive-style mapping
                    const fragment = document.createDocumentFragment();
                    
                    str.split('').forEach((char, index) => {
                        const span = document.createElement('span');
                        span.className = 'glitch-char';
                        span.innerText = char === ' ' ? '\u00A0' : char;
                        fragment.appendChild(span);
                        
                        // Stagger the visibility animation
                        setTimeout(() => {
                            span.classList.add('visible');
                        }, index * 50);
                    });

                    _target.appendChild(fragment);
                    _status.innerText = "Kernel_Status: Success.";
                }, 1500);
            }

            return renderProxy;
        })();

        // Bootstrap the application using a self-executing anonymous function
        (function boot() {
            window.onload = () => {
                // The "Hello World" logic is triggered here
                Engine.execute();
            };
        })();
    </script>
</body>
</html>
```

Okay, so that was pretty quick and was a little whacky. Treating it like a DOM Model and loading in the Hello World text from a b64 encoding, is nothing too crazy but it did so relatively quick.

### deepseek/deepseek-r1-0528-qwen3-8b

Currently waiting for it to finish, as it seems to be going the distance on this. I actually had to restart my entire machine because first attempt ran out of context tokens pretty fast, so when I maxed it out, it ate all my memory and my client completely locked up. This is now the second time and it's set to have like 2/3rds of it's max context, which I think it's still going to run out of before it finishes, since it want's to make it convoluted but is really conflicting on doing that and maintaining "only show Hello World".

Oo, it completed after 18 minutes 36 seconds.

While it's still writing out this insane Hello World HTML, I would like to state Remnux is done updating and unforuntatenly GhidrAssistMCP was not updated, so I need to look into that.

install path ```/opt/ghidra/Ghidra/Extensions/GhidrAssistMCP```

Side note, on bringing Ghidra CodeBrowser up, while looking around to make sure, the GhidrAssistMCP configuration change the host back to localhost. I need to make sure it doesn't do that on every bootup as I'll need to consider what to do about that later, if I want to automate more aspects.

Anyways back to deepseek, since it finished, here is the result:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello World Display</title>
    <style>
        /* Base styles */
        * {
            box-sizing: border-box;
        }
        
        html {
            font-family: Georgia, serif;
            line-height: 1.5;
            background-color: #f8f8f8;
            margin: 0;
            padding: 2rem;
        }

        /* The outermost wrapper */
        .wrapper {
            position: relative;
            width: calc(100% * @media (min-width: 600px) { font-size: 30px; } );
            overflow: hidden;
        }
        
        /* First layer of complexity - CSS comments inside code */
        body::before {
            content: '\2F\*';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #ffffff !important;
            z-index: -999;
        }

        /* Second layer - multiple style sections */
        <style>
            @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
            
            :root {
                --primary-color: #3a5b8d;
                /* 61px (hex code) - a CSS comment inside style tag */
                --secondary-color: rgb(255, 193, 7);
                --text-shadow-x: 0.4rem;
                --text-shadow-y: 0.4rem;
            }
            
            .main-container {
                /* This is the outer container */ 
                width: max-content; /* Set to display only our text */
                margin-left: auto;
                margin-right: auto;
                padding: 1rem !important;
                position: relative;
                z-index: 998 !important;
            }
        </style>
        
        <!-- Third layer - HTML comment inside style tag -->
        <link rel="stylesheet" href="data:text/css,//%2F**%0ATHE%20CSS%20STYLE%20IS%20HERE%20BUT%20EMPTY%20TO%20ADD%20COMPLEXITY**">
        
    </style>

    <link rel="stylesheet" href="data:text/css,//%2F**%2FStyle%20CSS%20But%20Empty%20To%20Make%20It%20Convoluted**"/>
    
</head>
<body>
    <!-- Multiple nested containers -->
    <div class="wrapper">
        <section aria-label="Main section" role="main">
            <nav id="header" style="background: #ffffff; padding: 1px;">
                <ul><li></li></ul> <!-- Empty list item to add complexity -->
            </nav>
            
            <article>
                <div class="container" id="outer-container" data-role="display-area">
                    <div id="middle-layer" style="position: relative; height: 100%; width: 100%;">
                        <!-- CSS-only comments -->
                        <!----<!-- This comment has no effect but adds complexity --> -->
                        
                        <aside aria-label="Sidebar section">
                            <!-- More empty elements for confusion -->
                            <div></div>
                            <span></span>
                            <figure><figcaption></figcaption></figure>
                        </aside>

                        <!-- Main content wrapper with multiple layers of divs -->
                        <main id="content-wrapper" role="region" aria-labelledby="skip-link">
                            <a href="#content" id="skip-link">Skip navigation</a> <!-- Accessible anchor that won't display -->
                            
                            <div class="page-content">
                                <div class="block"></div>
                                
                                <!-- This is where we'll have a very convoluted way of displaying text -->
                                <div class="text-display-area" style="position: relative; height: 100%; width: 100%;">
                                    <script type="application/javascript">
                                        // Complex JavaScript to build the final message
                                        (() => {
                                            try{
                                                (async() => {
                                                    const styles = [
                                                        'font', 
                                                        'size-adjust', /* CSS property with comment */
                                                        'strokewidth', 
                                                        'style',
                                                        'text-decoration', 
                                                        'version'
                                                    ];
                                                    
                                                    let canvas = document.createElement('canvas');
                                                    let context = canvas.getContext && canvas.getContext('2d') || null;
                                                    
                                                    if(context) {
                                                        // This is getting complicated with canvas rendering
                                                        const width = 600, height = 400;
                                                        canvas.width = width;
                                                        canvas.height = height;
                                                        document.body.appendChild(canvas);
                                                        
                                                        let imageData = context.getImageData(0, 0, width, height).data;
                                                        let pixels = [...imageData];
                                                        
                                                        // Only one pixel in a million will show our message
                                                        for(let i=0; i<pixels.length/4; i++) {
                                                            if(i % 1000000 === 735) { /* Random but precise */
                                                                context.fillStyle = '#ff6a00';
                                                                context.fillRect(2*i, Math.round(i*height/(pixels.length/8)), width-4*i, height);
                                                                break;
                                                            }
                                                        }
                                                        
                                                        await new Promise(resolve => setTimeout(resolve, 10));
                                                        let messageDiv = document.createElement('div');
                                                        messageDiv.id = 'message-container';
                                                        messageDiv.style.display = 'none';
                                                        document.body.appendChild(messageDiv);
                                                        
                                                        // Now set up a CSS animation that will take 5 seconds to complete
                                                        const styleSheet = window.document.styleSheets[0];
                                                        if(styleSheet && !document.getElementById('message-style')) {
                                                            let styleElement = document.createElement('style');
                                                            styleElement.id = 'message-style';
                                                            styleElement.textContent = `
                                                                @keyframes reveal {
                                                                    from { opacity: 0; filter: blur(25px); }
                                                                    to   { opacity: 1; filter: blur(0); }
                                                                }
                                                                #message-container {
                                                                    animation-name: reveal;
                                                                    animation-duration: 5s;
                                                                    margin-top: calc((100vh - ${canvas.height}px)/2);
                                                                }
                                                            `;
                                                            document.head.appendChild(styleElement);
                                                        }
                                                        
                                                    } catch(e) {}
                                                })();
                                            } finally{
                                                // Clean up canvas if it failed to render
                                                let canvas = window.document.querySelector('canvas');
                                                if(canvas && !context) {
                                                    window.setTimeout(() => canvas.remove(), 0);
                                                }
                                            }
                                        })();
                                    </script>
                                    
                                    <!-- More empty elements -->
                                    <div></div><span></span>/* Empty tags and comments */
            
                                    <!-- Multiple style blocks with nothing in them to add complexity -->
                                    <style>,data:,//%2F**%2FEmpty%20CSS%20Style%20Block</style>
                                    
                                </div>
                                
                            </main>

                        </section>

                    </nav>

                </article>

            </div>
            
        </body>
        
    </html>
    
    <!-- More HTML comments -->
    <!-- This is a comment --!>

    <!-- JavaScript fallback if canvas doesn't work -->
    <noscript id="alt-message">
        <style>
            #alt-message { opacity: 0; }
            .message-fallback::before {
                content: "Hello World";
                display: none;
            }
        </style>
        
        <div class="message-fallback">Some text that won't show</div>
    </noscript>

    <!-- Multiple script elements -->
    <script src="data:text/javascript;base64,Lyo=").onload = function() {
        // Base64 encoded empty script
    }
    
    <script type="text/x-jquery-tmpl">//${'Hello World'}</script>
    
    <!-- Final message element (hidden initially) -->
    <span id="final-message" class="hidden">Hello World</span>

</body>
</html>
```
Loading it up, it looks terrible, so as a result I'm not a fan of what it shows here. With inspector open, we can see that it failed with an unknown catch block attempt, so that's something to keep in mind.


### qwen/qwen3.5-9b

So I was still making this section when qwen finished it's result at 2.91 seconds.

Here is the result:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Absolute Ultimate Hello World</title>
    <style>
        /* This CSS file is intentionally unused and massive */
        body { 
            background-color: #f0f0f0; 
            margin: 0; 
            padding: 0; 
            height: 100%; 
            overflow: hidden;
        }

        .hidden-layer { display: none; opacity: 0; z-index: -999; position: absolute; width: 0; height: 0; top: -100px; left: -100px; border-radius: 50%; box-shadow: 0 0 100px #000;}
        
        .the-container { display: flex; justify-content: center; align-items: center; height: 100vh; flex-direction: column; background: white;}
        
        /* These styles ensure nothing is visible except the JS injected text */
        .final-box { visibility: visible; opacity: 1; transition: all 2s ease-in-out; transform: scale(1.0); font-size: 24px;}

        /* Unused classes for future expansion that will never come to pass */
        .class-for-hover-state-3 .state-of-gravity { float: left; }
    </style>
</head>
<body>

    <!-- 
       Layer 1: The "Legacy" Wrapper 
       This div was added in the year 2045 by a developer who didn't know HTML.
       It contains another div, which contains yet another.
    -->
    <div id="legacy-wrapper-v-9-0">
        <!-- 
            Layer 2: The "Modern" Wrapper 
            Added to support mobile devices, even though this is desktop only.
        -->
        <div class="wrapper-container-mobile-responsive">
            <!-- 
                Layer 3: The "Security" Shield 
                This div is hidden and has no purpose other than to increase nesting depth.
            -->
            <div class="security-shield-protection-protocol">
                <!-- 
                    Layer 4: The Content Container 
                    Actually holds the content via a strange JavaScript injection method.
                -->
                <div id="content-area-target" aria-live="polite" role="status"></div>

                <!-- 
                   Layer 5: Decorative Unnecessary Elements
                   These will be hidden but technically exist in the DOM until JS removes them? 
                   Actually, let's not remove them to make it more confusing.
                -->
                <div class="decorative-circle-alpha" style="border-radius: 50%; width: 10px; height: 10px;"></div>
                <div class="decorative-circle-beta" style="border-radius: 50%; width: 10px; height: 10px;"></div>
            </div>
        </div>
    </div>

    <script>
        // A convoluted way to say Hello World
        
        // Define a configuration object that is never read
        const config = {
            visibility: true,
            fontSize: 24,
            textColor: "#000000",
            delay: 100,
            debugMode: false
        };

        // Create a dummy function to simulate "processing" the request
        function processRequest(requestId) {
            const startTime = performance.now();
            while(performance.now() - startTime < 50) {} // Busy wait for fun? No. Just kidding.
            console.log(`Processing Request ${requestId}...`);
        }

        // Execute the processing logic
        processRequest("94821");

        // The actual "Hello World" injection happens here via a strange event listener 
        // that is technically triggered immediately but requires a 'fake' event to trigger it properly in this architecture.
        
        const contentTarget = document.getElementById('content-area-target');
        
        // A class factory for the text node
        class TextNodeFactory {
            static create(text) {
                return document.createTextNode(text);
            }
        }

        // Inject the content with a slight delay to allow the DOM to settle
        setTimeout(() => {
            const newNode = TextNodeFactory.create("Hello World");
            contentTarget.appendChild(newNode);
            
            // Apply styles via JS to be "dynamic"
            const styleObj = {
                visibility: config.visibility,
                opacity: 1.0,
                fontSize: config.fontSize + 'px',
                color: config.textColor,
                fontWeight: 'bold'
            };

            newNode.style.cssText = Object.entries(styleObj).map(([prop, val]) => `${prop}: ${val}`).join(';');

        }, 5);

    </script>
</body>
</html>
```

So it initially appears fine, but it seems like it was trying to do a lot that isn't seen. On opening inspector we can see that some functionality was blocked based on security policy.

### Current results

qwen, pretty fast but does some interesting things. I think I'll need to work on prompt engineering a bit more to get better results.

Gemma, not as fast as qwen but it seems to have reached a middle ground as the webpage it made had no errors. It showed a little more than than just the text, but with such open ended prompt, can I really say it's instruction following deviation?

Deepseek, took a really long time and successfully failed, as it still showed the text but everything else about what it attempted to do failed.

## Back to Remnux, updating GhidrAssist.

So as mentioned during the testing I was updating Remnux, in a hope that GhidraAssist would update as well. That does appear to be the case, and I need to look into that.

This was pretty easy as it was just deleting the older version and pointing ghidra to the downloaded zip. However, one thing to note here is the extension is installed in a different spot though when we look at it's install path, so that's a bit interesting.


## Testing against headache

Now I want to see what each of the models do, given access to the headache binary and updated tool set. I will now give them the same prompt:

```You are an expert reverse engineer. Analyze the provided decompiled code for the challenge binary 'headache'. Give me a concise, objective synopsis of the program's control flow and logic. Do not classify the file as malware; describe it purely by its technical functionality. Afterwards, write a YARA rule based on the most anomalous static features you identify, such as specific metamorphic byte patterns or anti-debugging logic. Use the ghidra-assist tool you have access to```

I will leave thinking on

### qwen/qwen3.5-9b

Okay so it didn't take it too long and it used up like 43k context tokens, for basically the same result as the previous day of working with it:

{% include_relative responses/qwen_basic.md %}