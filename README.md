# FIFA Worldcup 2018 - JSON data

__The data file is updated every 5 minutes (approximately) during live matches.__

## Usage

To use the data in your app, you can use this link:

`
https://raw.githubusercontent.com/broeneatsdinner/fifa-worldcup-JSON/master/data/world-cup-2018.json
`

Updated every 5 minutes

## Project Overview

fifa-worldcup-JSON stems from an idea originally found over at [Martin Århof](https://github.com/lsv/fifa-worldcup-2018)'s (lsv) "fifa-worldcup-2018" project. The structure of the JSON data is largely the same, with a few exceptions noted below. Expanding on the structure of the data, I wanted to build a friendly robot (scraper) that waits for games to start and then goes out into the world on a headless browser to "watch" every game, recording the statistics it finds into a JSON file.

The result of the robot's findings is a JSON file that sits in this GitHub repository, automatically being updated every 5 minutes (or so) during live matches. The data in the JSON file (listed above, and [here](https://raw.githubusercontent.com/broeneatsdinner/fifa-worldcup-JSON/master/data/world-cup-2018.json)) can be polled repeatedly by your own application, so you can use it either as a backend for your own API, or you can read through it in a human-like fashion to see the latest World Cup stats.

### Goals for Proof-of-Concept

- Update a JSON file with the latest World Cup scores
- Refresh the JSON every five minutes
	- (Randomly throttled so that the various websites our "robot" visits don't ban us for scraping)
- Only perform refreshes during game times (all times will be in UTC)
- __Don't sacrifice readability for the sake of saving a few bytes' data transfer.__ The JSON should be human-readable without having to look up id's and do computer(y) things like cross-referencing winners and losers.

### Process

- Script to:
	- Check current time on machine
	- Compare current time to game start times (both in UTC)
	- Scrape google.com / yahoo.com / fifa.com for latest scores at random intervals
	- Save scores (initially) and extra data like elapsed time, goal-scorers (secondary) to local JSON file
	- Diff local JSON file with GitHub JSON file
		- If differences exist, set a flag
		- If no differences exist, ignore update
	- Check if flag is set, then:
		- Automatically commit changes to JSON file
		- Set commit message with current time (in UTC)
		- Push commit to GitHub repository
	- Rinse and Repeat

#### Considerations

- A safe time to "watch" would be 3 hours (starting at game start, and allowing enough time to finish with stoppage time and penalties, if necessary).
	- The rules call for 90 minutes plus stoppage time. The break between halves is normally about 15 minutes. Extra time is used when there is a tie at the end of the normal time period. Total time from start to win is around 2 hours 10 minutes, unless extra time is needed, which is not uncommon.
- Technologies considered are GitHub, PHP, Node.js, Nightmare.js, Electron browser, and Cron jobs

## Data Structure

### stadiums

```
{
    id        Integer
    name      String
    city      String
    latitude  Float
    longitude Float
    image     String
}
```

_Sample_

```
{
    "id": 1,
    "name": "Luzhniki Stadium",
    "city": "Moscow",
    "latitude": 55.715765,
    "longitude": 37.5515217,
    "image": "https://upload.wikimedia.org/wikipedia/commons/e/e6/Luzhniki_Stadium%2C_Moscow.jpg"
}
```

### teams

```
{
    id                        Integer
    name                      String
    eliminated                Boolean
    eliminated_at_which_stage String ["groups", "round_16", "round_8", "round_4", "round_2_loser", "round_2"]
    fifa_code                 String
    iso2                      String
    flag                      String
    emoji                     String
    emoji_string              String (unescaped Unicode)
}
```

_Sample_

```
{
    "id": 1,
    "name": "Russia",
    "eliminated": true,
    "eliminated_at_which_stage": "round_16",
    "fifa_code": "RUS",
    "iso2": "ru",
    "flag": "https://upload.wikimedia.org/wikipedia/en/thumb/f/f3/Flag_of_Russia.svg/900px-Flag_of_Russia.png",
    "emoji": "flag-ru",
    "emoji_string": "🇷🇺"
}
```

### groups

```
String : Object
{
    name     String
    winner   String
    runnerup String
    matches  Array of Objects
        id           Integer
        type         String
        home_team_id Integer
        away_team_id Integer
        home_team    String
        away_team    String
        home_score   Integer
        away_score   Integer
        home_scorers String
        away_scorers String
        date         Datetime (ISO 8601 format)
        stadium_id   Integer
        channels     Array of Integers
        time_elapsed String
        finished     Boolean
        matchday     Integer

}
```

_Sample_

```
"a": {
    "name": "Group A",
    "winner": "Russia",
    "runnerup": "Uruguay"
    "matches": [
        {
            "id": 1,
            "type": "group",
            "home_team_id": 1,
            "away_team_id": 2,
            "home_team": "Russia",
            "away_team": "Saudi Arabia",
            "home_score": "5",
            "away_score": "0",
            "home_scorers": "A. Golovin 90'+4' D. Cheryshev 90'+1' A. Dzyuba 71' D. Cheryshev 43' Y. Gazinskiy 12' ",
            "away_scorers": null,
            "date": "2018-06-14T15:00:00+00:00",
            "stadium_id": 1,
            "channels": [
                4,
                6,
                13
            ],
            "time_elapsed": "Final",
            "finished": true,
            "matchday": 1
        }
    ]
}
```

### knockout

```
String : Object
{
    name     String
    matches  Array of Objects
        id           Integer
        type         String
        home_team    String
        away_team    String
        home_score   Integer
        away_score   Integer
        home_penalty Integer
        away_penalty Integer
        winner       String
        loser        String
        home_scorers String
        away_scorers String
        date         Datetime (ISO 8601 format)
        stadium_id   Integer
        channels     Array of Integers
        time_elapsed String
        finished     Boolean
        matchday     Integer

}
```

_Sample_

```
"round_16": {
    "name": "Round of 16",
    "matches": [
        {
            "id": 49,
            "type": "qualified",
            "home_team": "Group A winner",
            "away_team": "Group B runnerup",
            "home_score": null,
            "away_score": null,
            "home_penalty": null,
            "away_penalty": null,
            "winner": null,
            "loser": null,
            "home_scorers": null,
            "away_scorers": null,
            "date": "2018-06-30T14:00:00+00:00",
            "stadium_id": 11,
            "channels": [
                4,
                13
            ],
            "time_elapsed": null,
            "finished": false,
            "matchday": 4
        }
    ]
}
```

## Data Explanation

### Match types

#### group

_Pre-filled at tournament start_<br>
A regular group match.

#### qualified

_Calculated by winner and runnerup in each group_<br>
The `home_team` and `away_team` fields will be populated with the `winner` or `runnerup` of each group, as those positions are filled. These fields are strings, uniquely identifying a team by teams > name.

Prior to groups having a winner or having a runnerup, the `home_team` and `away_team` will be indicated by a placeholder string, e.g. "Group A winner", or "Group B runnerup".

#### winner

_Calculated by the winner of a particular match, as identified by match id > winner_<br>
The `home_team` and `away_team` fields will be populated with the `winner` of a particular match, when that winner position is filled. These fields are strings, uniquely identifying a team by teams > name.

Prior to a particular match being played (and thus, having a winner), the `home_team` and `away_team` will be indicated by a placeholder string, e.g. "match_id 49 winner", or "match_id 50 winner".

In the above scenario, "match_id 49 winner" indicates the `winner` of the match with `id` 49 will populate the field.

#### loser
##### Only used for Third place play-off

_Calculated by the loser of a particular match, as identified by match id > loser_<br>
The `home_team` and `away_team` fields will be populated with the `loser` of a particular match, when that loser position is filled. These fields are strings, uniquely identifying a team by teams > name.

Prior to a particular match being played (and thus, having a loser), the `home_team` and `away_team` will be indicated by a placeholder string, e.g. "match_id 61 loser", or "match_id 62 loser".

In the above scenario, "match_id 61 loser" indicates the `loser` of the match with `id` 61 will populate the field.

### Knockout Matches

Knockout matches occur after group matches have concluded, and they cannot end in a draw. Therefore a knockout match has the following data:

```
"home_score": null,
"away_score": null,
"home_penalty": null, // new, different than "group" matches
"away_penalty": null, // new, different than "group" matches
"winner": null,       // new, different than "group" matches
"loser": null,        // new, different than "group" matches
```

When a knockout match is completed without going to penalties, it may look like this:

```
"home_score": 0,
"away_score": 1,
"home_penalty": null,
"away_penalty": null,
"winner": "Sweden",
...
```

The `winner` field will have a team's name (string), identifying them by teams > name.<br>
The `loser` field will have a team's name (string), identifying them by teams > name.

If a knockout match goes to penalties, the result may look like this:

```
"home_score": 1,
"away_score": 1,
"home_penalty": 4,
"away_penalty": 5,
"winner": "Sweden",
...
```

## Key Differences

A few key differences to take note of, as this JSON file may differentiate from other data sources you have been using:

#### 1)

`home_team` is always a String, not an Integer, for example "Sweden", not 23.

`away_team` is always a String, not an Integer, for example "Costa Rica", not 19.

These changes were made to improve readability of the actual raw data.

***

#### 2)

`date` is always in __UTC__ time zone, in the ISO 8601 format, for example "2018-07-15T15:00:00+00:00".

These changes were made to ensure accurate date/time conversion in your own app(s), regardless of where your system is located, where you may be located, or where in Russia the match is being played.

***

#### 3)

`winner` and `loser` have been added to all __knockout__ matches in all rounds. The fields are Strings indicating the team's name.

These changes were made to improve readability of the raw data, and prevent small recursive lookups of losing team names in apps.

***

#### 4)

`eliminated` was added to the __teams__ data, and is a Boolean. This field indicates whether a given team has been eliminated from advancing to the next stage in the tournament.

`eliminated_at_which_stage` was added to the __teams__ data, and is a String. This field indicates at which stage the team was eliminated, and will always be either:

- `"groups"`
- `"round_16"`
- `"round_8"`
- `"round_4"`
- `"round_2_loser"`
- `"round_2"`

These changes were made as a request.

***
