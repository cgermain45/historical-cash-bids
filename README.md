# Historical Cash Bids

An internal tool for viewing a grain location's cash bids on a past date, laid out like a normal bid display and exportable to CSV or Excel.

## Using it

1. Open the tool's GitHub Pages link.
2. Enter your Barchart OnDemand API key. The tool won't pull anything without one.
3. Enter one or more location IDs, separated by commas or spaces. Location IDs come from the admin.
4. Pick a date and click **Go**.
5. Use **Export CSV** or **Export Excel** to save the table.

The tool remembers the last location IDs and commodity roots you used.

## What you get

One row per bid that was active at the close on the selected date, grouped by location:

| Column | Meaning |
|---|---|
| Commodity | The commodity the bid is for |
| Delivery start / Delivery end | Exact delivery window for the bid |
| Futures month | The futures contract the bid is priced against |
| Futures | That contract's daily close (settlement) on the selected date |
| Basis (¢) | The bid's closing basis, in cents |
| Cash price | The bid's closing cash price, in dollars |

Hovering over a row shows its Barchart cash bid symbol. Exports also include the date, location ID, elevator, location and symbol.

## How it works

The tool runs entirely in your browser and calls Barchart OnDemand directly. No server is involved.

1. **Find every bid the location has had.** For each location ID and commodity root, it calls `getGrainInstruments` (`instrumentType=bidCash`). This returns every cash bid symbol for that location, including expired delivery periods, along with the delivery dates and the futures contract each bid is priced against.
2. **Narrow to bids that could have existed on the date.** Symbols whose delivery period ended before the selected date are dropped.
3. **Check which bids actually had a price that day.** For each remaining symbol it calls `getHistory` (`type=daily`) for the selected date. A bid counts as active only if it has a closing value that day. Delivery dates alone aren't enough, because a bid can be posted later than its delivery window opens.
4. **Add basis and futures.** For each active bid it fetches the matching basis symbol, and it fetches the close for each futures contract involved.
5. **Build the table** and sort it by location, commodity and delivery start.

Cash bid symbols follow this pattern:

```
ZCPV26-87738-18585.CM
│ ││  │     │      └ cash commodity suffix
│ ││  │     └ customer commodity ID
│ ││  └ location ID
│ │└ delivery month and year (V = Oct, 26 = 2026)
│ └ P = cash price  (B = basis)
└ commodity root (ZC = corn)
```

Results are cached for the session, so running the same date again is faster.

## Things to know

- **Trading days only.** Weekends and market holidays have no closing values, so they return nothing. Use the previous trading day.
- **Commodity roots.** The default list is `ZC, ZS, ZW, KE, MW` (corn, soybeans and three wheat classes). A root a location doesn't bid returns nothing and is noted above the table. Add other roots in the field if needed.
- **"Close," not "settlement," for cash bids.** The futures column is the exchange close. The cash price and basis are the last values of Barchart's cash bid series for that day. They aren't official settled prices.
- **Basis units.** The basis symbol's units weren't confirmed when this was built. The tool chooses between cents and dollars by checking which one agrees with cash minus futures.
- **Speed.** Each possible bid is one history request, run six at a time. A location with a long bid history can take 20–40 seconds the first time.
- **Missing data.** Some cash bid symbols have no stored history in Barchart. Those bids won't appear, and nothing flags them.

## API key handling

- The key is never stored in the code or the repository.
- It's kept in the browser only for the current tab, unless you tick **Remember on this computer**, which saves it in that browser.
- Saved keys are stored as plain text in the browser. Don't tick Remember on shared computers.
- Browser storage is shared by every GitHub Pages site under the same account, so keep this repo under an account that doesn't host unrelated sites.
- **Forget key** removes it.

## Hosting

The tool must be opened from a web address such as GitHub Pages. Opening `index.html` directly from your computer fails with "Couldn't reach Barchart," because the browser blocks calls to Barchart from a local file.

To update the tool, replace `index.html` in the repository, wait a minute for GitHub Pages to redeploy, then hard-refresh the page with Ctrl+Shift+R (Cmd+Shift+R on Mac).

## Troubleshooting

| Message | Likely cause |
|---|---|
| Couldn't reach Barchart | Opened as a local file, or a browser extension is blocking the request. Use the hosted link and try an incognito window. |
| The API key was rejected | Wrong or expired key. |
| getGrainInstruments returned HTTP 400 | A request parameter Barchart doesn't accept. The message after the code is Barchart's own reason. |
| No end-of-day bids found | A weekend or holiday, a wrong location ID, or no bids for those commodity roots on that date. |
| A location shows "no bid symbols found" | That location has no cash bids for the roots entered. |
