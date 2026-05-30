---
title: Spotify Playlist Automation
description: A practical guide to building AI-assisted Spotify playlists from plain text genre lists.
pubDatetime: 2026-05-30T05:37:57Z
tags: [spotify, oauth, playlists, llm, automation]
---

Spotify playlists look simple from the outside: a name, a cover, and a list of songs.
But once you try to create playlists programmatically, the interesting parts appear
quickly. You need authentication, token refresh, search, track selection, playlist
creation, and error handling. If you want the playlist to feel good, you also need
taste.

This post walks through a practical design for building Spotify playlists from plain
text files, with an LLM helping choose the canonical version of each song.

## The Goal

The goal is not just "create a playlist." The goal is:

1. Write playlists as simple text files.
2. Search Spotify for each song.
3. Pick the best, most representative version of each result.
4. Create real Spotify playlists.
5. Add the selected track URIs.
6. Report anything that failed without stopping the whole run.

The text files can be organized by genre:

```text
playlists/
  blues.txt
  country.txt
  disco-funk.txt
  electronic-new-wave.txt
  folk-singer-songwriter.txt
  hip-hop-golden-age.txt
  jazz.txt
  pop-classics.txt
  rock-classics.txt
  soul-rnb.txt
```

Each file uses one song per line:

```text
The Beatles - Hey Jude
Michael Jackson - Billie Jean
Aretha Franklin - Respect
```

This format is deliberately boring. It is easy to read, easy to edit, easy to diff,
and easy to feed into a script.

## The Basic Architecture

The whole workflow can be understood as a small pipeline:

```text
playlist file
  -> song line
  -> Spotify search query
  -> Spotify search candidates
  -> LLM chooses best candidate
  -> selected Spotify track URI
  -> Spotify playlist API
```

The main pieces are:

| Piece | Responsibility |
| --- | --- |
| Token refresh script | Use the saved refresh token to get a fresh access token |
| Song search module | Convert a query into one chosen Spotify track URI |
| Playlist builder | Read playlist files, create playlists, and add tracks |
| LLM picker | Choose the canonical track from Spotify search results |

Keeping these responsibilities separate makes the system easier to reason about.
Search is not playlist creation. Token refresh is not search. Playlist file parsing is
not LLM judgment.

## Authentication: Access Tokens and Refresh Tokens

Spotify Web API calls use OAuth. In practice, this means the app needs an access
token when it calls Spotify. Access tokens expire, so the app also stores a refresh
token locally. The refresh token is used to request a new access token when needed.

The local token flow looks like this:

```text
read refresh_token
  -> POST Spotify token endpoint
  -> receive access_token
  -> save access_token back to local token file
```

The important security rule is simple: do not commit the real token file or secret
environment file. Keep those local and ignored by git.

## Spotify Search Is Not Canonical Selection

Spotify search is a catalog search endpoint. If you ask for:

```text
Hey Jude
```

Spotify may return several things:

- the original Beatles recording
- a remaster
- a compilation version
- a cover
- a live version
- an unrelated fuzzy match

That is normal. Search APIs return candidates, not taste.

A better query can help:

```text
track:"Hey Jude" artist:"The Beatles"
```

This narrows the candidate set before any LLM decision happens. The script can build
this query from a plain text line:

```text
The Beatles - Hey Jude
```

into:

```text
track:"Hey Jude" artist:"The Beatles"
```

That helps Spotify do a better first pass.

## Why Use an LLM?

Even with a good query, there can still be multiple plausible versions. For a famous
song, a remaster on a compilation may be better than a random cover. But sometimes
the first result is not the one a listener would expect.

The LLM is used as a judge. It receives a compact list of Spotify candidates, not the
entire raw API response. The compact candidate list can include:

- candidate index
- track name
- artist names
- album name
- album type
- release date
- duration
- explicit flag
- playability
- popularity, if present
- ISRC, if present
- Spotify URL
- Spotify URI

The prompt asks the model to choose the "real, classic, original, most popular, or
most representative" version. It also tells the model to avoid covers, karaoke,
tribute versions, live versions, remixes, instrumentals, and unrelated fuzzy matches.

The LLM must return structured JSON:

```json
{
  "selected_index": 1,
  "selected_uri": "spotify:track:...",
  "reason": "Canonical recording by the original artist."
}
```

The script then validates the response. The selected index must exist, and the
selected URI must exactly match the URI in the selected candidate. This validation is
important: the LLM is allowed to choose, but it is not allowed to invent.

## The Reusable Function

The useful abstraction is:

```python
def pick_song_uri(query: str) -> str:
    ...
```

This function does one job: turn a Spotify search query into one selected track URI.

Internally, it:

1. Reads the saved access token.
2. Calls Spotify search.
3. Compacts the candidate list.
4. Sends candidates to the LLM.
5. Validates the LLM choice.
6. Returns one URI.

The return value is exactly what the playlist API needs:

```text
spotify:track:0aym2LBJBk9DAYuHHutrIl
```

That one function becomes the bridge between "human song list" and "Spotify playlist
item."

## Creating Playlists From Files

The playlist builder reads every `.txt` file in the playlist directory. The filename
becomes the Spotify playlist name, with a prefix:

```text
blues.txt -> codex-blues
rock-classics.txt -> codex-rock-classics
soul-rnb.txt -> codex-soul-rnb
```

For each file, the builder:

1. Reads non-empty, non-comment lines.
2. Converts each line into a Spotify query.
3. Calls `pick_song_uri(query)`.
4. Collects successful track URIs.
5. Remembers failed songs.
6. Creates a private Spotify playlist.
7. Adds the collected URIs to that playlist.

Spotify accepts up to 100 items per add-items request, so the script adds tracks in
chunks. That does not matter much for 25-song genre playlists, but it makes the
script ready for larger lists.

## Failure Handling

Playlist automation should not be fragile. If one song fails, the whole run should
not collapse.

A good failure policy is:

- if token refresh fails, stop
- if required configuration is missing, stop
- if one song cannot be resolved, remember it and continue
- if a playlist has zero resolved songs, skip creating it
- after everything finishes, print all failed songs

The final report can look like this:

```text
Summary
Playlists created: 10
Songs added: 244
Failed songs: 6
Failed playlists: 0

Failed songs:
- playlist=codex-blues | song='Artist - Song' | query='track:"Song" artist:"Artist"' | error='...'
```

This style is friendly to long-running work. You get the playlists that can be built,
and you get a precise repair list for the songs that need attention.

## Handling Expired Access Tokens

Spotify access tokens expire. The builder should refresh once before doing any real
playlist work. It should also handle a `401 Unauthorized` from Spotify by refreshing
once and retrying the operation.

That gives the script two layers of protection:

1. proactive refresh at startup
2. reactive refresh if Spotify rejects a later request

This is especially useful when a playlist build makes many search calls and LLM calls.
The script may run long enough that a token becomes stale.

## Why Plain Text Playlists Work Well

Using text files for playlists has a few nice properties:

- they are easy to version in git
- they can be reviewed like code
- they can be edited without a UI
- they are portable across tools
- they separate curation from API mechanics

The playlist file is the source of taste. The scripts are only machinery.

For example, a `jazz.txt` file can be curated slowly over time. Later, the automation
can rebuild or recreate the Spotify playlist from that file.

## The Bigger Pattern

This project is really an example of a broader pattern:

```text
human-curated source data
  -> deterministic API search
  -> LLM judgment for ambiguity
  -> strict validation
  -> side-effectful API action
  -> final report
```

The LLM is not replacing the system. It is sitting at the one place where ambiguity
is real: which search result is the best cultural match?

Everything around it should be boring and strict:

- JSON in
- JSON out
- exact URI validation
- skipped failures
- no secret printing
- no silent guessing

That is the sweet spot for this kind of tool.

## Future Improvements

The first version can stay simple, but there are several natural improvements:

- make the playlist folder configurable
- allow command-line selection of one playlist file
- cache successful song-to-URI matches
- detect duplicate URIs before adding tracks
- support dry-run mode
- support updating existing playlists instead of always creating new ones
- write failed songs to a report file
- include market configuration for Spotify availability
- let the LLM return a confidence score

The most valuable next improvement is probably caching. Once the system has resolved
`The Beatles - Hey Jude` to a URI, it should not need to spend another Spotify search
and LLM call for the same line unless the user asks for a refresh.

## Closing Thought

Spotify playlist automation is not only about saving clicks. It is about separating
three different kinds of work:

- curation: choosing the songs
- judgment: choosing the right recording
- execution: creating the playlist

Plain text files are good for curation. Spotify is good for catalog operations. An
LLM is useful for judgment when search results are messy. Put together carefully, the
result is a small but satisfying music tool: human taste, API execution, and machine
assistance each doing the part they are good at.
