# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

My version scores each song against a listener's stated taste. It looks at mood, genre, energy, valence, and acousticness. Matches on mood and genre earn flat points. Energy and valence earn points based on how close they are, not just yes or no. I also spent time testing the system on purpose, trying weird or conflicting profiles to see where the scoring breaks down or quietly favors certain users.

---

## How The System Works

Real-world recommenders like Spotify, YouTube Music, and Apple Music rarely rely on a single signal. They blend **collaborative filtering** (recommending a song because users with similar listening history enjoyed it) with **content-based filtering** (recommending a song because its own attributes, like tempo or mood, match what a listener already likes). Collaborative filtering is good at surfacing songs a listener wouldn't have picked for themselves, but it needs a large base of other users' behavior and struggles with brand-new songs or users. My version prioritizes **content-based filtering only**: it has no notion of other users, so it scores each song purely on how closely its attributes (genre, mood, energy, valence, and acousticness) match a single user's stated taste profile. Each numeric feature is scored by how close it is to the user's target value rather than by how high or low it is, and those per-feature scores combine into one weighted total per song. This makes every recommendation fully explainable. I can point to the exact features that drove a match, but it also means my system will never suggest something outside a user's stated preferences the way a collaborative system might.

Some prompts to answer:

- What features does each `Song` use in your system
  - For example: genre, mood, energy, tempo
- What information does your `UserProfile` store
- How does your `Recommender` compute a score for each song
- How do you choose which songs to recommend

You can include a simple diagram or bullet list if helpful.

Features each song use in my system:
- genre
- id
- mood
- energy
- valence
- acousticness

Features/information that UserProfile stores:
- favorite_genre
- favorite_mood
- target_energy
- likes_acoustic
- target_valence

### Finalized Algorithm Recipe

Each song is scored out of a **10-point maximum**, split across five weighted components. Mood is weighted highest because it reflects listening *intent* ("I want something chill right now") more directly than a genre label does, and the dataset shows genre is a noisy proxy for mood (e.g. `lofi` and `ambient` both mean "chill"). The two categorical fields (mood, genre) are worth more combined than the numeric ones because an exact taste match is a stronger signal than closeness on a single continuous attribute.

| Component | Type | Weight | Formula |
|---|---|---|---|
| Mood match | categorical | 3 pts | `3 if the mode of the song == the user's favorite mood, else 0` |
| Genre match | categorical | 2 pts | `2 if the song's genre == the user's favorite genre, else 0` |
| Energy closeness | numeric | 2 pts | `2 * (1 - abs(song.energy - user.target_energy))` |
| Valence closeness | numeric | 2 pts | `2 * (1 - abs(song.valence - user.target_valence))` |
| Acousticness alignment | numeric | 1 pt | `song.acousticness if user.likes_acoustic else (1 - song.acousticness)` |

`score_song()` sums these five values and returns `(total_score, reasons)`, where `reasons` is the list of components that fired (e.g. `"mood matched (happy)"`, `"energy close: 0.82 vs target 0.80"`). `recommend_songs()` sorts all songs by `total_score` descending and returns the top `k`. Ties are broken by keeping the original CSV order (stable sort), so no song is silently favored beyond what its score earns.

**Expected biases:**
- **Mood/genre lock-in.** Exact categorical matches dominate the score (5 of 10 points), so the system will rarely surface a song outside the user's stated mood or genre even when its numeric profile is a near-perfect fit.
- **Adjacent-label penalty.** Musically similar but differently-labeled genres (`lofi` vs `ambient`) get zero credit, since the genre match is exact-string, not semantic.
- **Boolean acousticness collapses nuance.** A single `likes_acoustic` flag scores a mildly acoustic-averse user identically to a strongly acoustic-averse one.
- **Hand-picked weights, not learned ones.** The 3/2/2/2/1 split reflects my own judgment rather than measured behavior, which is a source of designer bias baked into the ranking.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

CLI output:
```
User profile - mood: happy, genre: pop
Top recommendations:

Sunrise City - Score: 6.96
Because: mood matched (happy); genre matched (pop); energy close: 0.82 vs target 0.80

Rooftop Lights - Score: 4.92
Because: mood matched (happy); energy close: 0.76 vs target 0.80

Gym Hero - Score: 3.74
Because: genre matched (pop); energy close: 0.93 vs target 0.80

Night Drive Loop - Score: 1.90
Because: energy close: 0.75 vs target 0.80

Block Party Anthem - Score: 1.88
Because: energy close: 0.86 vs target 0.80
--------------------------------------------------------------------------------
User profile - mood: happy, genre: High-energy pop
Top recommendations:

Sunrise City - Score: 4.94
Because: mood matched (happy); energy close: 0.82 vs target 0.85

Rooftop Lights - Score: 4.82
Because: mood matched (happy); energy close: 0.76 vs target 0.85

Block Party Anthem - Score: 1.98
Because: energy close: 0.86 vs target 0.85

Storm Runner - Score: 1.88
Because: energy close: 0.91 vs target 0.85

Gym Hero - Score: 1.84
Because: energy close: 0.93 vs target 0.85
--------------------------------------------------------------------------------
User profile - mood: chill, genre: chill lofi
Top recommendations:

Midnight Coding - Score: 4.96
Because: mood matched (chill); energy close: 0.42 vs target 0.40

Library Rain - Score: 4.90
Because: mood matched (chill); energy close: 0.35 vs target 0.40

Spacewalk Thoughts - Score: 4.76
Because: mood matched (chill); energy close: 0.28 vs target 0.40

Focus Flow - Score: 2.00
Because: energy close: 0.40 vs target 0.40

Coffee Shop Stories - Score: 1.94
Because: energy close: 0.37 vs target 0.40
--------------------------------------------------------------------------------
User profile - mood: intense, genre: deep intense rock
Top recommendations:

Gym Hero - Score: 4.96
Because: mood matched (intense); energy close: 0.93 vs target 0.95

Storm Runner - Score: 4.92
Because: mood matched (intense); energy close: 0.91 vs target 0.95

Neon Pulse Overdrive - Score: 2.00
Because: energy close: 0.95 vs target 0.95

Crown of Thunder - Score: 1.96
Because: energy close: 0.97 vs target 0.95

Block Party Anthem - Score: 1.82
Because: energy close: 0.86 vs target 0.95
``` 



**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

I turned off the mood check to see how much it really mattered. The rankings changed completely. Songs with a perfect energy match jumped ahead of songs that used to win on mood alone.

I tried a profile with genre text that didn't exactly match the catalog, like "High-energy pop" instead of "pop." The genre points never fired, since matching is exact text only. The recommender quietly fell back to energy as the deciding factor, without saying so.

I tried profiles with conflicting preferences, like a "sad" mood paired with high energy. The system doesn't notice the conflict. It just adds up the points like normal and returns whatever score comes out.

I counted how many songs share each mood and genre in my catalog. Most moods and genres only have one matching song. That means most listeners can only ever get one "perfect" match in the whole catalog, no matter how the numbers work out.

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

It only works on a small, 18-song catalog. It does not understand lyrics, language, or the era a song is from. Genre and mood only match on exact text, so close-enough words score nothing. Most moods and genres only have one song each, so niche listeners get weak results. There's also a gap in the energy data between 0.55 and 0.74, so medium-energy listeners don't have many good options. The weights I used (3/2/2/2/1) are my own guesses, not based on real listener behavior. And since it has no idea about other users, it can never surprise someone with a song they wouldn't have picked for themselves.


---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this

Building this taught me that a recommender is really just a set of rules someone decided on, and those rules always carry some bias. My scoring math was correct, but the weights I picked and the shape of my catalog still tilted results toward common tastes like "happy pop" and away from niche ones. Nothing in the code was broken, but the results still weren't fair to every kind of listener.

I also learned that testing matters as much as building. Turning off the mood check, trying mismatched text, and testing weird or conflicting profiles showed me problems I never would have found just by reading my own code. It changed how I think about real recommender apps: an algorithm can look objective on paper while still quietly favoring whoever the data and weights happen to be built around.



