# Troubleshooting

## The caller is not asked for the magic word

Check that:

- Twilio is using the production webhook URL
- the n8n workflow is active
- Twilio uses `POST`
- the production webhook is publicly reachable over HTTPS
- **Incoming Intercom Call** is configured to respond through a response node

Check Twilio call logs and the most recent n8n execution for webhook or TwiML errors.

## The prompt repeats or loops

The speech `<Gather>` must send its result back to the workflow as a follow-up request. Confirm that its `action` resolves to the production webhook URL and that the second request contains `body.SpeechResult`.

Reverse proxies can affect the host or protocol used to build the callback URL. Confirm that n8n receives the correct `host` and `x-forwarded-proto` headers.

## The correct magic word is rejected

Check the execution data from **Extract Transcription** and compare `transcription` with `magicWord`.

Possible causes:

- speech recognition produced different words
- the intercom microphone distorted the magic word
- background noise affected transcription
- the configured language does not match the speaker
- the magic word was changed in one workflow version but not another

Choose a short, distinctive magic word. Avoid uncommon names, individual letters, or words that sound similar over a phone line.

## An incorrect magic word is accepted

The public workflow uses a case-insensitive `contains` comparison. This is convenient, but a longer sentence containing the magic word will also match.

For a stricter rule, normalize punctuation and whitespace, then compare the entire transcription for equality. You can also add schedules, retry limits, or a second condition.

## The magic word matches, but the door does not unlock

Possible causes:

- wrong unlock digit
- the intercom does not recognize Twilio's DTMF tone
- the building rejects the Twilio number
- the intercom needs a longer delay before the digit
- the call ends before the digit is played

The unlock response uses a leading lowercase `w`, which tells Twilio to pause briefly before sending the digit. Add more `w` characters if the intercom needs a longer delay, for example:

```xml
<Play digits="www6"/>
```

Each lowercase `w` adds a 0.5-second pause.

## Twilio shows error 12100

Twilio error `12100` commonly indicates invalid TwiML. Check that:

- the response begins with `<Response>`
- the response ends with `</Response>`
- the XML is valid
- no extra text appears before the XML declaration
- the response uses `Content-Type: text/xml`

## The webhook works in test mode but not production

Make sure:

- the workflow is active
- Twilio uses the production URL rather than the test URL
- the webhook path has not changed
- the n8n instance is publicly reachable

## A normal phone works, but the building intercom does not

A normal call only verifies Twilio, speech recognition, and n8n. It does not prove the building will accept the VoIP number or the DTMF tone.

Check:

- the building assigned the correct number to the unit
- the number uses the format expected by the intercom
- the intercom is allowed to call that number's region
- the building accepts VoIP numbers
- the unlock digit and delay are correct

## Webhook validation fails

Twilio signature validation depends on the exact URL and request parameters. Common causes include:

- validating a different public URL than the one Twilio requested
- URL decoding or re-encoding before validation
- excluding form parameters from the signature calculation
- using the wrong Twilio Auth Token
- modifying the request in a proxy before validation

See [Twilio's webhook security documentation](https://www.twilio.com/docs/usage/webhooks/webhooks-security).
