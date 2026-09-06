# DRIVE AGENT — Browse while voice continues

This is a separate release. Earlier GitHub Pages links and the existing Luma voice app are unchanged.

## Preview links

- [Dark chat](https://tootooki.github.io/driveagent-browse-with-voice-06sep2026/demo.html?workspace=chat&design=dark)
- [Light chat](https://tootooki.github.io/driveagent-browse-with-voice-06sep2026/demo.html?workspace=chat&design=light)
- [Illustrated chat](https://tootooki.github.io/driveagent-browse-with-voice-06sep2026/demo.html?workspace=chat&design=studio)
- [Accounting and the voice popup](https://tootooki.github.io/driveagent-browse-with-voice-06sep2026/demo.html?workspace=accounting&design=light)

The homepage ENTER DEMO button opens Accounting. The selected design follows the logo and homepage category links. The main header, expandable menu and resizable Accounting/PPC columns are preserved.

## Chat page

The three designs share one implementation and conversation history. They use solid backgrounds with no fades, duplicate chat title, extra top controls, plus button or microphone. A typing field and send/stop button remain at the bottom. Replies stream into the existing message element; the input is not recreated or refocused. Each message has copy and read-aloud buttons. Typed replies stay silent unless the user presses read.

There are 50 strategy rooms, MAINCHAT and any Accounting product conversations. Names are capitalized without spaces and have small image/emoji tiles. The selected room is white, and the other rooms are black. The rail can be dragged from icon-only width to a wider list. Each room retains its own draft and messages, and begins with a silent AI greeting. Suggestions submit a real prompt. The illustrated design adds original local SVG artwork.

## Voice popup

The floating button opens a white panel covering the entire workspace below the existing header, including the sheet and product images. The same floating button stays in place and becomes X. There is no separate CHAT heading. Three circular controls provide settings, live voice and a voice message.

The microphone records until Stop is pressed, and Stop submits the recording. At the 60-second limit recording stops but still waits for Stop before submission. Microphone tracks are released as soon as recording ends. Live voice detects the end of each spoken turn, transcribes it, generates a reply, speaks it, then listens again. It is an alternating listen/speak loop, not simultaneous listening during playback. Quiet turns are discarded. Closing the panel, opening sheet menus and switching Accounting/PPC workspace tabs leave recording, replies and playback running. Reopening the floating button returns to the active conversation without restarting the microphone. Explicitly stopping voice, changing to a different conversation, leaving/hiding the browser page or a permission/service error ends the session. Read-aloud never requests microphone access.

Settings include voice, playback speed, automatic reading of voice replies and the conversation switcher. Microphone permission is requested only after a voice control is pressed. The panel uses black and white. While it is closed and voice is active, the floating black circle shows colorful fog: cyan/violet for listening, pink/violet for speaking and warmer motion while preparing a reply. A microphone/playback audio meter drives its intensity. The animation stays clipped inside the circle and respects reduced-motion preferences. Tap the circle to reopen the controls; Stop Live ends the live session, and microphone Stop sends its recording. No global settings or message popups were added.

## AI service

The dedicated Worker is `https://driveagent-chat-api-06sep2026.luma-voice-svlad92.workers.dev`. It is separate from the older voice app. Both typed and spoken replies use Cloudflare Workers AI Llama 3.1 8B Instruct Fast. Whisper Large V3 Turbo transcribes recorded speech, and Aura generates audio. No local model download or paid subscription upgrade was needed.

Only the selected room's recent message history and room context are sent for a reply. Recordings and text requested for speech are sent to the hosted AI service. Chat history stays in browser local storage; the Worker does not persist it. No API credential is embedded in the website. Requests have bounded sizes, timeouts and simple per-client/service rate limits. Cloudflare quota or temporary service failures appear as a retryable error; the site never substitutes a scripted answer for a failed AI reply.

This release provides real AI conversation, but it is not connected to live Amazon data or action execution. The sheet remains a prototype. The assistant is instructed not to claim account access or invent completed actions.

Local source for the dedicated service is in `worker/index.mjs` and `worker/wrangler.jsonc`. Deploy with authenticated Wrangler separately from the static GitHub Pages release.

## Validation

This release adds regression checks for live recording across panel close/reopen, Accounting/PPC/Chat navigation, microphone Stop-to-send after browsing, replies and playback while the panel is closed, room continuity and explicit media cleanup. A local browser fixture supplies simulated audio for visual checks without recording the room.

Automated coverage includes streamed replies, interrupted streams, retries without duplicate user messages, room isolation, draft persistence, copy/read controls, keyboard geometry, menu state, column resizing, microphone Stop-to-send, delayed permission results, voice activity detection and cleanup. Browser checks cover narrow phone and desktop layouts. The composer uses a true 16px font and retained focus, with existing VisualViewport handling preserved.

A synthetic audio test exercised the deployed transcription → AI reply → speech pipeline: 3.13 seconds total, with first response text about 0.25 seconds after transcription. That is one short controlled test, not a latency guarantee. The generated MP3 was valid, and browser read-aloud completed. Physical iPhone keyboard and microphone behavior has not been verified on a device in this release.

Run `npm test` for unit checks. Run the complete suite with `DOLCE_QA_JSDOM=/path/to/jsdom node --max-old-space-size=4096 --test tests/*.test.mjs tests/*.integration.mjs`. Publish the committed new release with `npm run publish`. The publisher refuses to overwrite an existing nonempty release repository.

References: [Workers AI models](https://developers.cloudflare.com/workers-ai/models/), [Whisper Large V3 Turbo](https://developers.cloudflare.com/workers-ai/models/whisper-large-v3-turbo/), [Aura](https://developers.cloudflare.com/workers-ai/models/aura-1/), [VisualViewport](https://developer.mozilla.org/en-US/docs/Web/API/VisualViewport).
