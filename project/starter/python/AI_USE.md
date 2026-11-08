# AI Use Disclosure — PA1

_Generative AI is allowed on PA1 as a tutor, reviewer, debugging partner and
helper with tooling. It is not allowed to be the author of your work. See the PA1
brief, "AI, Authorship and You". Fill in every section. "None" is an acceptable
answer only if it is true._

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [x] Yes, as described below.

## 2. Statement (the syllabus form)

> The author (Drake Mashburn) acknowledges the utilization of Google Gemini Flash, a language model
> developed by Google LLC, in the preparation of this assignment. Google Gemini Flash
> was employed in the following manner(s) within this assignment: 
> debuging
> establishing understanding of python libraries.
> giving ideas on how to proceed from walls
> other manners that are allowed within the assingment

> The author (Connor Reagan) acknowledges the use of Copilot, an everyday AI agent 
> developed by Microsoft, in the preparation of this assignment. Copilot 
> was used in the following ways within this assignment:
> Debugging
> Understanding a few questions
## 3. Log

_One row per substantive interaction. Summaries are fine; you do not need to paste
whole conversations, but keep them in case you are asked._

https://share.gemini.google/UqJm1hFo8EEw

| # | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
log is listed above in the form of a gemini link. I (Drake Mashburn) in general implemented most of what the ai said, I rejected ideas within the chat itself, see chat logs. Ran final product within the selfcheck.py to see if things were correct

* In server.py, I asked Copilot to assist me on some trivial bugs found in the file. The bugs were related to the errors and that I didn't realize you can't raise a number as an exception. I used that to change the raises to ValueErrors. I checked by running the etp_selfcheck.py file.
* In client.py, I asked Copilot why one of the function calls was not working and why etp needed to be used in certain places. I ran into a problem in the client where some of the functions needed etp while some of the constants did not. Reviewing the conversation from Copilot, I rectified the changes by adding etp in the function calls and constants from etp.py. Much like server.py, I also used the etp_selfcheck.py file.
* In evidence/ANALYSIS.md, I had it clarify some details on some of the questions that I was unsure on how to do. I asked Copilot to give me a brief description of how to do the question without telling me the answer. I used the instructions to help me find and work out the specific answers. I had to go back and change `server_to_client_payload_bytes` after doing one of the problems. This time, I went back and read the specifications from the ETP1_Specification.pdf, which really helped in some of the explanations I provided.

## 4. Where AI-produced material remains

transport_tcp.py, server.py (within the constraints given in the assingment)
client.py (within the constraints given in the assignment)

## 5. One thing the AI got wrong — or how you checked it

when doing the utf encoding, it initially did not specify that the .encode it use was a python library based encode and not the etf encode, which I thought it was, I had to ask for clarification on that and it was able to clarify.

Originally when I was doing client.py, it told me that using the raise for the constants was ok. As I started the debugging phase, Copilot realized that raising constants represented as values was not ok.


