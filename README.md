# Mock Players

This is a GitHub repost of @Misiur's `ut_mock_players` library. This purely exists because it hasn't been hosted on GitHub and [sampctl](https://github.com/Southclaws/sampctl) uses GitHub repositories for Pawn packages.

If you would like to host this on your own account, open an issue here and I'll close the repository.

Original documentation from the [forum post](http://forum.sa-mp.com/showthread.php?t=490767) below:

---

## Player mocking system for unit tests

y_testing is really powerful library, but most of you people won't use it ever - therefore ut_mock_players isn't for you. For the rest of you folks, sometimes you might need to simulate a lot of users on your server at once. Some people will use NPC's to do that, other will hire cheap Mexican labour. However, I'm not clever enough to do that, so I've started to overwrite functions from a_players. There is a metric alot of them, and I've done only few so far - this gives you idea how to do more of them, and even with this limited toolset, it's pretty helpful.

## Dependencies

Of course `y_testing`, and `y_utils`. Later on there will be `y_iterate` as well, so you can test with less than `MAX_PLAYERS` at once.

> Note from Southclaws:
>
> Side note: installation with sampctl will take care of dependencies

## Usage

```pawn
#define RUN_TESTS
#include <ut_mock_players>

main() {
    new
        tests,
        fails,
        func[33];
    Testing_Run(tests, fails, func);
}

public OnPlayerConnect(playerid)
{
    printf("Player %d connected!", playerid);
    SetPlayerScore(playerid, GetPlayerScore(playerid) + 25);
    return 1;
}

Test:Connection()
{
    //Randomly selected number
    new
        playerid = 256, score = GetPlayerScore(playerid);

    ASSERT(score == 0);
    printf("Player score before: %d", score);
    OnPlayerConnect(playerid);

    score = GetPlayerScore(playerid);
    ASSERT(score == 25);
    printf("Player score after: %d", score);
}

#include <ut_mock_players>
```

Wait, what, why is it included twice? Well, explanation is simple - you might want to run tests aimed at live players. Then, you want your normal functions back - and to do this, simply include this file again.

Very important note! It is essential to compile with suppresed warnings 203-206. "Why?" you might ask. Normally when you grab, for example, user position using GetPlayerPos, like:

```pawn
new Float:X, Float:Y, Float:Z;
GetPlayerPos(playerid, X, Y, Z);
```

All of your variables are considered being "used" by the compiler, even if you don't touch Z variable at all. Behind the scenes, ut_mock_players substitutes function call with standard assignmnent, resulting in code:

```pawn
new Float:X, Float:Y, Float:Z;
(X = MockPlayer[playerid][mu_pos][0], Y = MockPlayer[playerid][mu_pos][1], Z = MockPlayer[playerid][mu_pos][2]);
```

Now, if you don't use Z variable in your code, for compiler it is unused. I could rewrite code to use standard function hook, but assuming your code itself doesn't have any synax errors (passing function instead of variable reference), suppressing warning has lesser cost. Also, some functions are completely overridden, like IsPlayerConnected, which returns constant 1. That creates code like

```pawn
if(1) { /*(...)*/ }
```

Causing "expression has no effect" warnings. (I might change that to grab MockPlayer[playerid][mu_connected], but that's for future reference)

Example compiler call (used by me):

```pawn
pawncc "$file" -;+ -v2 -w203 -w204 -w205 -w206 -d3
```

> Note from Southclaws:
>
> with sampctl, you simply update your `pawn.json`/`pawn.yaml` to contain an `args` section in one of the `builds` objects:
>
> ```json
> {
>     "builds": [
>         {
>             "name": "tests",
>             "args": ["-w203", "-w204", "-w205", "-w206"]
>         }
>     ]
> }
> ```

## Installation

Simply add to your `pawn.json` and include:

```json
{
    "dependencies": ["Southclaws/ut_mock_players"]
}
```

```pawn
#include <ut_mock_players>
```

## Testing

To test, simply run the package:

```bash
sampctl package run
```

Then connect to `localhost:7777`.
