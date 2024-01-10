# OSTW Button Handler Module

## A relatively light-weight [OSTW](https://github.com/ItsDeltin/Overwatch-Script-To-Workshop) module used to precisely manage button presses from a player (or bot).

### What's the point?
If you have a mode that relies a lot on various key-press-activated commands, this might make it easier to manage. Such as "one click for X" or "two clicks for Y".

### What can it do:
- Track multiple, chained clicks from a player - If your script requires a user to press a button a certain number of times to trigger an action.
- Track how long a button has been held for - Allows you to accurately track how long a button has currently been held for.
- Precision - Tracking the hold length is accurate up to about ~15-30ms. Ping doesn't seem to play much of a part in this either.

### Multi-Clicks
Multi-Clicks track when a button is pressed multiple times in quick succession. You can use this to trigger different events with the same button. Similar to how single and double click works. The default is set to 300ms. Presses within ~300ms are tracked as a multi-click.

___

## How To Use

#### Make your rule and then use the following macros as your conditions:

- [<b>Press Count</b>](#button-press-count--button-hold-length) - Track the number of times a button has been pressed.<br>
`Number buttonPressCount(Player player, EButtonInput button)`<br>
  - `player`: The player you want to track.
  - `button`: The button you want to track. Follows the same structure as the OSTW `Button` enum.

  Example: `If (buttonHoldLength(player, button) >= 3)`<br>

<br>

- [<b>Hold Length</b>](#button-press-count--button-hold-length) - Track the length of time a button has been held for (in seconds).<br>
`Number buttonHoldLength(Player player, EButtonInput button)`<br>
  - `player`: The player you want to track.
  - `button`: The button you want to track. Follows the same structure as the OSTW `Button` enum.

  Example: `If (buttonHoldLength(player, button) >= 1.5)`<br>

<br>

- [<b>Press & Hold</b>](#button-press--hold) - Combines the two above into one condition/macro.<br>
`Boolean buttonPressAndHold(Player player, EButtonInput button, Number clickCount, Number holdLength)`<br>
  - `player`: The player you want to track.
  - `button`: The button you want to track. Follows the same structure as the OSTW `Button` enum.
  - `clickCount`: How many times the button should be pressed.
  - `holdLength`: How long the button should be held for.
  

  Example: `If (buttonPressAndHold(player, button, 3, 1.5))`<br>

  ___

## Examples

### Button Press Count & Button Hold Length

OSTW:

```hs
rule: "Double-Click Interact"
Event.OngoingPlayer
if (buttonPressCount(EventPlayer(), EButtonInput.Interact) == 2)
{
    SmallMessage(EventPlayer(), "You double-clicked Interact!");
}

rule: "Triple-Click Melee & hold for 2.5 seconds"
Event.OngoingPlayer
if (buttonPressCount(EventPlayer(), EButtonInput.Melee) == 3)
if (buttonHoldLength(EventPlayer(), EButtonInput.Melee) >= 2.5)
{
    SmallMessage(EventPlayer(), "You double-clicked Interact and held it for 2.5 seconds!");
}
```

Workshop Output:

```hs
rule("Double-Click Interact")
{
    event{Ongoing - Each Player;All;All;}

    conditions
    {
        X Component Of(Value In Array(Player Variable(Event Player, buttonHandler), 5)) == 2;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Interact!"));
    }
}

rule("Triple-Click Melee & hold for 2.5 seconds")
{
    event{Ongoing - Each Player;All;All;}

    conditions
    {
        X Component Of(Value In Array(Player Variable(Event Player, buttonHandler), 8)) == 3;
        Subtract(Global Variable(BH_gameClock), Y Component Of(Value In Array(Player Variable(Event Player, buttonHandler), 8))) >= 2.5;
    }
    actions
    {
        Small Message(Event Player, Custom String("You triple-clicked Melee and held it for 2.5 seconds!"));
    }
}
```

### Button Press & Hold

OSTW:

```hs
rule: "Double-Click Primary Fire and hold for 5 seconds"
Event.OngoingPlayer
if (buttonPressAndHold(EventPlayer(), EButtonInput.PrimaryFire, 2, 5))
{
    SmallMessage(EventPlayer(), "You double-clicked Primary Fire and held it for 5 seconds!");
}
```

Workshop Output:

```hs
rule("Double-Click Primary Fire and hold for 5 seconds")
{
    event{Ongoing - Each Player;All;All;}

    conditions
    {
        And(X Component Of(First Of(Player Variable(Event Player, buttonHandler))), Compare(Subtract(Global Variable(BH_gameClock), Y Component Of(First Of(Player Variable(Event Player, buttonHandler)))), >=, 5)) == True;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Primary Fire and held it for 5 seconds!"));
    }
}
```

  ___

### Pros:

1. It's pretty accurate within like ~30ms. You can use an external macro program to accurately test timings.
2. Very simple to use in an OSTW project.
2. It does stuff.

### Cons:

1. Specifying a button's hold-length to be below the set multi-click requirement isn't reliable. The design of this module means that it thinks the button is always held for **at least** the multi-click timeout period.
   - You can get around this, by adding the Workshop `IsButtonHeld` value to your conditions.
2. Trying to trigger an action based on a multi-click will also trigger those bound to a lesser click-requirement; unless your hold length requirement is greater than the multi-click timeout. **This is on my TODO list to fix.**<br>
**For example:**
   - I want to add 1 to a number for a single-click;
   - And add 2 to a number for a double-click.
  
   When double-clicking, both the single-click action and the double-click action will be triggered if the specified hold length is less than the multi-click timeout.

3. *Slightly minor, but expected* - Trying to rapidly trigger an action with a button press wont work due to the multi-click timeout. 
   - For example: If you want to quickly trigger a double-click action multiple times, you have to wait the duration of multi-click timeout before you can double-click again.

4. I made it.