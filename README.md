# Morse Trainer

A single-file, offline browser trainer for sending and receiving Morse code with a straight key, built around the Koch method.

**Live demo:** https://YOUR-USERNAME.github.io/morse-trainer/ (placeholder, update after enabling GitHub Pages)

## Run it locally

Open `index.html` in a browser. That is the whole install: no build step, no dependencies, no network requests. Everything is inline in one file and all settings persist in `localStorage`.

Send with the spacebar on a desktop, or the large on-screen key on a phone. Tap or press any key once to start; that first gesture unlocks audio.

## How it works

You are shown a target character. Above it, the ideal pattern is drawn as green bars: a dit is one unit wide, a dah is three, with one-unit gaps. Below it, what you actually key is drawn as blue bars, scaled to real elapsed time, so you see your rhythm error and not only right or wrong. A held press shorter than two dit lengths is a dit; anything longer is a dah.

Three practice modes, chosen in the settings panel:

- **Character**: one character at a time. The attempt is judged as soon as you have keyed as many elements as the target has. After a correct attempt the next character loads on its own after a short pause.
- **Spell**: a whole word, keyed one character at a time. Each character gets its own bar rows. A character ends when you stay silent for a character gap. Correct characters lock in and stay on screen, and the next character loads automatically; after the last one the next word loads.
- **Transcribe**: the app plays a word with standard spacing and you type what you heard. You get the correct answer and a per-character diff.

## Assist levels

The assist control has four presets. Each one sets the four toggles beneath it; changing any single toggle by hand relabels the preset as **Custom**.

| Level | Letter shown | Green pattern | Target audio |
|---|---|---|---|
| 1 Guided | yes | always | auto-plays, replay any time |
| 2 Recall | yes | hidden until you miss twice | auto-plays, replay any time |
| 3 Ear | no | hidden | auto-plays, replay any time |
| 4 Blind | no | hidden | plays once, no replay until the attempt ends |

Whenever the green pattern is hidden, the retry ladder applies:

1. First miss: the offending blue bar flashes red, the row clears, and you try again. Nothing is revealed.
2. Second miss: the green pattern is revealed and stays up while you key it again.
3. The character is then marked as missed, so the weighting serves it again sooner. The pattern hides again for the next character.

If the pattern is already visible (level 1), a miss just clears the row and you try again with the pattern still up.

## Timing

All timing constants are in the `TIMING` object at the top of the script in `index.html`.

- `ditMs = 1200 / wpm`. Default 15 wpm, adjustable from 5 to 30.
- A press is a dit if held for less than two dits, otherwise a dah.
- In word modes, silence longer than two dits ends a character and silence longer than five dits ends a word, with a 400 ms floor on both so slow beginners are not cut off.
- After a correct character the app pauses for `autoAdvanceMs` (default 600 ms, adjustable from 200 to 1500 ms with the "Pause between characters" slider) before loading the next one. Key presses that start during that pause are ignored so you cannot key into a stale target. A finished word pauses for `wordAdvanceMs` (800 ms) before the next word.
- Sidetone is a 600 Hz sine from the Web Audio API. One oscillator feeds two gain gates, one for the straight key and one for target playback, so keying and playback never share automation. Every edge is a 5 ms linear ramp. Playback is scheduled up front as absolute-time automation from a running cursor, and any in-flight playback is cancelled before a new one starts.

## Curriculum

Characters unlock in Koch order:

```
K M R S U A P T L O W I . N J E F 0 Y V , G 5 / Q 9 Z H 3 8 B ? 4 2 7 C 1 D 6 X
```

Level N unlocks the first N characters. Characters are drawn at random from the unlocked set, weighted toward recent misses. Word modes only use words made entirely of unlocked characters. If no word fits, the trainer serves random groups until you raise the level.

The stat line tracks attempts, accuracy, and streak for the session. Expand the per-character table to see accuracy for each unlocked character.

## Adding characters or word lists

Everything lives in `index.html`. The script is divided into numbered, commented sections.

**Characters.** Add the character and its pattern to the `MORSE` table in section 2, then insert it where you want it in the `KOCH_ORDER` string in the same section. The level slider reads its maximum from the length of that list, so nothing else needs to change. Prosigns can be added the same way using a single-character key.

**Word lists.** The built-in lists are `COMMON_WORDS` and `HAM_WORDS` in section 3. Both are plain whitespace-separated strings, so add words anywhere in them. Words must use only characters present in `MORSE`.

**Your own words without editing code.** Paste a list into the custom word list box in settings, one word per line. It overrides the built-in list, is filtered to the unlocked characters, and persists in `localStorage`. Clear the box to go back to the built-in list.

## License

MIT. See `LICENSE`.
