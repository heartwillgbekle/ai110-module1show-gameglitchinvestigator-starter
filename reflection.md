# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
The game looked normal
- List at least two concrete bugs you noticed at the start  
  (for example: "the secret number kept changing" or "the hints were backwards").

a. The hints were off. It said go lower and I kept going lower and the answer I got was way higher than the secret number. I expected it to tell me to go higher or lower depending on the value I enter and the value of the secret number and not just keep telling me to go lower.

b. The range of output I could put in was off. It said from I to 100 but it kept telling me to go higher and I entered values greater than 100 an it still kept saying "go higher". What I expected this to do is stop when it reached the end of the range.

c. The ranges given in the different mode do not correspond well with the secret number. The secret number in hard mode is sometimes higher than the range given for hard mode and that applies to the other modes as well. Also I think the different ranges have been swapped. I expect the hard mode to have a rather huge range and the easy mode will have a rather smaller range. and also the secret number should not be over the range limit

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?

I used Claude Code (Anthropic's Claude) as my primary AI tool throughout this project. I used it to read and analyze the buggy code, identify the root causes of each bug, and apply targeted fixes.

- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).

Claude correctly identified that the hint messages in check_guess were reversed — when the guess was too high, the code returned "Go HIGHER!" instead of "Go LOWER!". Claude pointed me to app.py:L37-L40 and explained the logic error. I verified this by reading the function myself and confirming that guess > secret should direct the player to go lower, not higher. Running the game and entering a number I knew was above the secret confirmed the fix worked.

- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).

At first, Claude only flagged the swapped hint strings as the cause of bug (a) and (b). I initially assumed fixing that one line would resolve all hint issues. However, testing still showed inconsistent hints on some turns. Going back with Claude, we discovered a second hidden bug on every even-numbered attempt, the secret number was silently converted to a string, causing Python to compare an integer guess against a string using lexicographic ordering (e.g., 50 > "9" fails unexpectedly). The first analysis missed this because the string conversion was not in the check_guess function itself but in the calling code in app.py.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?

I decided a bug was fixed by first reading the corrected code to confirm the logic made sense, then manually playing the game and testing the specific scenario that was broken. For example, after fixing the reversed hints I entered a number I knew was above the secret and verified the game now said "Go LOWER!" instead of "Go HIGHER!". Only when the game behaved correctly for several test cases in a row did I consider the fix done.

- Describe at least one test you ran (manual or using pytest)
  and what it showed you about your code.

I ran the existing pytest suite in test_game_logic.py, which contains three tests: test_winning_guess, test_guess_too_high, and test_guess_too_low. These tests call check_guess directly with known inputs and assert the correct outcome string. Running them confirmed that the comparison logic was right — check_guess(60, 50) returned "Too High" and check_guess(40, 50) returned "Too Low" as expected. The tests were also useful for catching the string-vs-int bug because I could trace exactly which inputs produced wrong outcomes without having to click through the UI every time.

- Did AI help you design or understand any tests? How?
Yes. Claude explained that the existing tests only checked the outcome string (e.g., "Too High") but not the hint message (e.g., "Go LOWER!"), so they would pass even if the hint text was still wrong. That helped me understand the difference between testing the outcome label and testing the user-facing message, and reminded me to verify both manually in the running app even after pytest passed.

---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.

In the original app, the secret number was generated with a plain random.randint(low, high) call that ran every time the script executed. Streamlit re-runs the entire Python file from top to bottom on every user interaction — clicking a button, typing in a box, anything. So each click triggered a fresh random.randint, producing a completely different secret number with no memory of what it was before.

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
Imagine every time you click a button on a webpage, the server throws away the whole page and rebuilds it from scratch using the same recipe. That's a Streamlit rerun — the full script runs again from line 1. Session state is like a sticky notepad that survives those rebuilds: anything you write to st.session_state stays there across reruns, so values like the secret number, the attempt count, and the score aren't lost every time the user interacts with the app.

- What change did you make that finally gave the game a stable secret number?
The fix was wrapping the secret generation in a guard: if "secret" not in st.session_state: st.session_state.secret = random.randint(low, high). This means the random number is only generated once — the very first time the app loads. On every subsequent rerun, the "if" condition is False so the existing secret is left untouched, giving the game a stable target for the entire session.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.

Reading the code myself before accepting any AI fix. Every time Claude suggested a change, I opened the file and traced the logic line by line to confirm the explanation made sense. That habit caught the string-vs-int bug that the first AI pass missed — I wouldn't have found it if I had just applied the suggested fix and moved on. Going forward I want to make "read it before you merge it" a default instinct on any AI-assisted code change.

- What is one thing you would do differently next time you work with AI on a coding task?

I would ask AI to explain the entire relevant code path, not just the suspicious function. In this project I initially described only the symptom ("hints are wrong") and Claude focused on check_guess in isolation. The real cause was partly in the calling code in app.py. Next time I will share more surrounding context upfront and ask "are there any other places in the file that could affect this behavior?" before assuming the first fix is complete.

- In one or two sentences, describe how this project changed the way you think about AI generated code.

This project taught me that AI-generated code can look completely correct at a glance — the game ran, produced output, and even had reasonable structure — while hiding subtle logical bugs that only show up under specific conditions. I now treat AI output as a first draft that needs the same skeptical review I would give any untested code, not as a finished product.
