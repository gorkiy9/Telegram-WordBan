# Word Ban: hide a word you don't want to read in a chat

Some words you would rather not see: a colleague's endless "deadline", a spoiler, a name, a slur. Word Ban lets you
ban a word in one chat, for yourself only. Incoming messages with it keep coming, but the word is blacked out or
replaced with your own text or emoji. The sender and the server know nothing about it.

> Preliminary demo. The demo video is not recorded yet.

## What it does

### 1. Ban a word from a message

Long-press an incoming message, select the word, and choose **Ban a Word** in the selection menu.

### 2. The ban sheet

- The word, large, previewing the choice below it.
- A horizontal selector of how messages show it, **Visible** by default:
  - **Visible**: the word is not banned; there is no ban button (for a banned word, the button lifts the ban);
  - **Censored**: a strip of paper is pressed over the word from left to right, its free end lifted and bending
    until it lies flat, slightly tilted, with a soft shadow, with the same motion as on messages in the chat;
  - **Replace**: a field for your own text or emoji. Until you type, the word shows in the placeholder colour; then
    it moves back behind your text (smaller, blurred, fading downwards), and what you type appears in front of it
    character by character, without stretching the letters already shown.
- Once Censored or Replace is picked, **Ban Word** appears at the bottom and, to its right, how long the ban lasts,
  picked on a wheel: 1 Day, 1 Week, 1 Month, 3 Months, 6 Months, 1 Year or Forever.
- After the ban, a toast with Undo.

### 3. Messages with a banned word

- Every incoming message in the chat with the word (whole word, any case: "deadline", "Deadline", "DEADLINE" and
  "DeAdLiNe" are one word) covers it with the same strip or replaces it at once, old messages included; strips on messages already on screen are pressed on with the same motion.
- **Similar Words** (a switch in the sheet, on by default) also catches the word stretched ("Yooou" for "Yo"), its
  forms ("deadlines", "дедлайнами"; the system lemmatizer from NaturalLanguage, with common Russian and English endings
  for words it does not know) and its beginning stretched ("Йооо" for "Йоу": at least two letters and two thirds of the
  word, the last letter repeated). The switch's footer shows an example of each, made from the word being banned.
- The message's status line says **banned word** (**banned words** when it has more than one), next to the time, the way it says "edited". An edited message shows both: **banned word, edited 12:00**.
- The message's context menu has **Show Word** (for this message; **Hide Word** puts it back; **Show Words** / **Hide Words** for several) and **Unban "word"** for each banned word.

## How it works

- **Built from the client.** The sheet is `SheetComponent` with `TabSelectorComponent`, `ListTextFieldItemComponent`,
  `ListSwitchItemComponent`, `ButtonComponent` and `GlassBarButtonComponent`; the duration wheel is the client's
  `ChatTimerScreen` (as for auto-delete; a new `singleLinePickerItems` option shows "1 Week" and "Forever" in one
  font), the Similar Words row gets the input-field fill through a new optional `backgroundColor` of
  `ListSwitchItemComponent`, the toasts are `UndoOverlayController`. Colours and fonts come from `PresentationTheme`.
- **Display only.** `ChatMessageTextBubbleContentNode` applies the chat's bans to an incoming message's text before
  layout (`wordBanApply`: the replacement with entity ranges shifted, or the ranges to black out, which the bubble
  covers with bars from the text layout's range rects in the message's text colour). The stored
  message is never changed; copy, reply and forward use the original text.
- **No server changes.** Bans are kept on the device (`WordBanStore`, `UserDefaults`), per chat, with an end date. A real
  version would sync them across devices:

  | Feature | Needed API |
  | --- | --- |
  | Bans per chat, synced | `messages.getWordBans(peer)`, `messages.setWordBan(peer, word, mode, replacement, until)`, `messages.deleteWordBan(peer, word)` |
  | Bans on other devices | `updateWordBans(peer)` |

- **The demo build.** The demo is recorded on a separate offline build, so no real data can reach it. It is on only when
  the bundle id contains `WordBanDemo` (`isWordBanDemoBuild`): the app signs in to a local account without a server,
  points all data centres at a closed local port, and fills the chats from `TelegramUI/Sources/WordBanDemo.swift`.
  Launching with `-WordBanDemoReset YES` restores the chats and clears every ban. A regular build never enables any of
  this.

## Applying the change

`changes.patch` is the difference from the official Telegram-iOS master at commit `24638bc69b` (Telegram 13.2). From a
checkout of that commit (31 files, +2 312 −9):

```
git apply --check changes.patch
git apply changes.patch
```

Key files: `submodules/TelegramUI/Components/Chat/ChatWordBan/` (store and matching),
`WordBanSimilar.swift` there (similar words), `submodules/TelegramUI/Components/Chat/WordBanScreen/` (the sheet), `ChatControllerWordBan.swift` and
`ChatInterfaceStateContextMenus.swift` in `submodules/TelegramUI/Sources/`, the selection menu item in
`TextSelectionNode.swift`, and the display in `ChatMessageTextBubbleContentNode.swift`.

## Limitations

- Bans live on one device.
- Only the message bubble is filtered: the chat list preview, notifications and reply quotes still show the word.
