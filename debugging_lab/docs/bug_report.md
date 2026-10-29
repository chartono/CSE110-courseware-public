# Lemon Clicker Bug Report

**Name:**
**Tickets solved:** <!-- e.g. 1, 2, 3, 5, 6 (at least five) --> 5, 4

Copy the template below once for each ticket you solved. 

---

## Ticket <5>: <Leaderboard dates show "undefined">

### Actual cause

<!-- Which file and function was the bug in, and what was wrong with the code?
     Explain why that code produced the behavior described in the ticket. -->

**File / function:**
server/index.ts in type LeaderboardEntry and app.post("/api/leaderboard", ...)

**What was wrong:**
A property naming mismatch between the backend Express server and the frontend client. The Express server saved and return the time using the "time" variable, but ResultsScreenModel and ResultsScreenView use "timestamp" instead. Because entry.timestamp was evaluated on an object containing { score, time }, it evaluated to undefined, which is what displayed on the screen.

### Fix

<!-- What did you change, and why does that change fix the problem? -->
I changed the variable name in ResultsScreenModel and ResultsScreenView to "time" to match the Express server variable. This ensures that the frontend can correctly read the time form the backend server.

### How did you find it?

#### Hypothesis testing

**What hypothesis ultimately led you to identify the buggy code, and why?**

#### Backward tracing

<!-- Start from the symptom you saw and work backwards, one step at a time,
     until you reach the buggy line. For each step, note where you looked
     and what you saw there. Add steps as needed. -->

1. **Symptom:** <!-- what you saw, e.g. text on screen, console error --> Top scores entry in the Game Over screen displayed 18 - undefined
2. **Traced back to:** <!-- where that came from, e.g. a function, a request in the Network tab, a server log --> ResultsScreenView.ts line rendering ${entry.score} - ${entry.timestamp}, indicating entry.timestamp evaluated to undefined.
3. **Traced back to:** ResultsScreenController.submitScoreToServer(), which received the raw JSON array from POST /api/leaderboard.
4. **Buggy code:** <!-- the file, function and line where the trace ended --> server/index.ts, where type LeaderboardEntry and app.post("/api/leaderboard") created entries with { score, time } instead of { score, timestamp }

<!-- It is okay to use your judgement here in limiting the number of traceback steps. We want to understand your 
    reasoning for how you discovered where the bug was -->

### Confirming the fix

<!-- How did you confirm the problem is gone? -->

Played the game, got a score, and verified the Game Over screen. The entry rendered with both the score and the correct date string without showing undefined.

---

## Ticket <4>: <Stuck on the Game Over screen>

### Actual cause

<!-- Which file and function was the bug in, and what was wrong with the code?
     Explain why that code produced the behavior described in the ticket. -->

**File / function:**
endGame() function in GameScreenController.ts

**What was wrong:**
The functioned called stopTimer but did not include the brackets. Because the parentheses were missing, the function was never executed and setInterval continued ticking every second in the background. Each tick saw timeRemaining <= 0 and triggered gameOver() again, which repeatedly replayed the game-over sound, submitted duplicate scores to /api/leaderboard, and forced the view back to the results screen.

### Fix

<!-- What did you change, and why does that change fix the problem? -->
I changed the variable name in ResultsScreenModel and ResultsScreenView to "time" to match the Express server variable. This ensures that the frontend can correctly read the time form the backend server.

### How did you find it?

#### Hypothesis testing

**What hypothesis ultimately led you to identify the buggy code, and why?**

#### Backward tracing

<!-- Start from the symptom you saw and work backwards, one step at a time,
     until you reach the buggy line. For each step, note where you looked
     and what you saw there. Add steps as needed. -->

1. **Symptom:** <!-- what you saw, e.g. text on screen, console error --> The game-over audio replays every second and duplicate scores are submitted to the leaderboard continuously.
2. **Traced back to:** <!-- where that came from, e.g. a function, a request in the Network tab, a server log --> Multiple recurring calls to ResultsScreenController.showResultsFromServer(), which is triggered by gameOver() in GameScreenController.ts.
3. **Traced back to:** The timer tick callback in GameScreenController, which checks if (timeRemaining <= 0) every second and invokes this.gameOver().
4. **Buggy code:** <!-- the file, function and line where the trace ended --> GameScreenController.ts, inside gameOver(), where this.stopTimer was referenced without the parentheses ()

<!-- It is okay to use your judgement here in limiting the number of traceback steps. We want to understand your 
    reasoning for how you discovered where the bug was -->

### Confirming the fix

<!-- How did you confirm the problem is gone? -->

Played a game, and stayed on the Game Over screen for several seconds. The game-over sound played exactly once, only one score entry was posted to the leaderboard, and clicking "Play Again" successfully switched to the start screen without switching back.

---

## Ticket <6>: <My high score disappeared from the leaderboard>

### Actual cause

<!-- Which file and function was the bug in, and what was wrong with the code?
     Explain why that code produced the behavior described in the ticket. -->

**File / function:**
server/index.ts in the app.post("/api/leaderboard", ...) route handler

**What was wrong:**
The sorting comparator used to rank the scores was sorted in ascending order rather than descending order.

### Fix

<!-- What did you change, and why does that change fix the problem? -->
I changed the sort comparator from 'a.score - b.score' to 'b.score - a.score'.

### How did you find it?

#### Hypothesis testing

**What hypothesis ultimately led you to identify the buggy code, and why?**

#### Backward tracing

<!-- Start from the symptom you saw and work backwards, one step at a time,
     until you reach the buggy line. For each step, note where you looked
     and what you saw there. Add steps as needed. -->

1. **Symptom:** <!-- what you saw, e.g. text on screen, console error --> The lowest scores were ranked at the top of the leaderboard, while the higher ones were at the bottom.
2. **Traced back to:** <!-- where that came from, e.g. a function, a request in the Network tab, a server log --> The JSON returned by 'POST /api/leaderboard', which ordered entries from lowest score to highest score.
3. **Traced back to:** 'saveLeaderboard(topEntries)' and 'entries.slice(0, MAX_LEADERBOARD_ENTRIES)' in 'server/index.ts'
4. **Buggy code:** <!-- the file, function and line where the trace ended --> 'server/index.ts' inside 'app.post("/api/leaderboard", ...)', line 'entries.sort((a, b) => a.score - b.score);'.

<!-- It is okay to use your judgement here in limiting the number of traceback steps. We want to understand your 
    reasoning for how you discovered where the bug was -->

### Confirming the fix

<!-- How did you confirm the problem is gone? -->

Played a game and got a new high score, checked the results screen and confirmed that the top score was the new highscore and that the other scores were properly sorted in descending order.

---

## Ticket <1>: <Stats always say "Stats unavailable">

### Actual cause

<!-- Which file and function was the bug in, and what was wrong with the code?
     Explain why that code produced the behavior described in the ticket. -->

**File / function:**
MenuScreenController.ts in the loadStatsFromServer() method

**What was wrong:**
There was an endpoint URL typo in MenuScreenController.ts, fetched "/api/stat" instead of "/api/stats".

### Fix

<!-- What did you change, and why does that change fix the problem? -->
I changed fixed the typo to fetch "/api/stats".

### How did you find it?

#### Hypothesis testing

**What hypothesis ultimately led you to identify the buggy code, and why?**
The network request fetching statistics was failing with a 404 error due to a route path mismatch between the client and server. To test this, I opened the browser devtools and observed a failed request: GET http://localhost:xxxx/api/stats 404 (Not Found). Comparing this URL with server/index.ts confirmed that the server registers app.get("/api/stats", ...), meaning the client was querying a non-existent endpoint /api/stat.

#### Backward tracing

<!-- Start from the symptom you saw and work backwards, one step at a time,
     until you reach the buggy line. For each step, note where you looked
     and what you saw there. Add steps as needed. -->

1. **Symptom:** <!-- what you saw, e.g. text on screen, console error --> 
2. **Traced back to:** <!-- where that came from, e.g. a function, a request in the Network tab, a server log --> 
3. **Traced back to:** 
4. **Buggy code:** <!-- the file, function and line where the trace ended --> 

<!-- It is okay to use your judgement here in limiting the number of traceback steps. We want to understand your 
    reasoning for how you discovered where the bug was -->

### Confirming the fix

<!-- How did you confirm the problem is gone? -->

Refreshed the webpage and confirmed that stats are now displayed instead of "Stats unavailable".
