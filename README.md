# Smart Buzzer n8n Workflow

A free DIY workflow that answers an apartment buzzer, asks the visitor to say the magic word, and sends the DTMF unlock digit when the magic word matches.

I used this version myself for a few weeks, then set it up for a few friends. Eventually, I turned it into [BuzzerBee](https://buzzerbee.app?utm_source=github&utm_medium=repo&utm_campaign=smart_buzzer_n8n).

If you already use n8n and are comfortable managing Twilio and webhooks, you can build and run it yourself with this repository. So far, it has worked with every phone-based intercom we have tried.

## How it works

This is designed for any intercom that dials a resident's cell phone number and unlocks when the resident presses a digit.

```text
Visitor buzzes
  -> intercom calls a Twilio number
  -> Twilio asks for the magic word
  -> Twilio transcribes the response
  -> n8n checks the magic word
  -> If it matches: Twilio sends the DTMF unlock digit
  -> If no match: Twilio rejects the call

No additional building hardware is required.

## Compatibility

The workflow is designed for intercoms that:

- dial a cell phone number
- unlock when you press a digit
- accept a Twilio number and DTMF tones

It will not work with an intercom that only uses a closed video system or proprietary app.

## What's included

- [`workflows/smart-buzzer-n8n.json`](workflows/smart-buzzer-n8n.json): sanitized n8n workflow
- [`docs/setup.md`](docs/setup.md): Twilio and n8n setup
- [`docs/troubleshooting.md`](docs/troubleshooting.md): transcription, DTMF, and compatibility issues

## Quick start

1. Import the workflow into n8n.
2. Open **Configuration** and set the magic word and unlock digit.
3. Copy the production URL from **Incoming Intercom Call**.
4. Configure the Twilio number's incoming Voice webhook to send a `POST` request to that URL.
5. Activate the workflow.
6. Test the correct magic word, an incorrect magic word, and silence.
7. Test the complete call path from the building intercom.

The example configuration is:

```text
magicWord: open sesame
unlockDigit: 6
```

Replace both values before using the workflow. Do not use the public example magic word for real access.

## Important limitations

- The workflow does not validate Twilio's `X-Twilio-Signature`.
- Speech recognition can mishear the magic word. Numbers are the most reliable (e.g 123), but it's more fun to make your friends say silly words. 
- There are no failed-attempt limits, schedules, logs, or alerts.

Review these limitations before connecting the workflow to a real intercom.

## Cost

The workflow is free under the MIT License. Running it requires a Twilio phone number, Twilio Voice and speech-recognition usage, and an n8n instance.

## Prefer a hosted version?

I later built [BuzzerBee](https://buzzerbee.app?utm_source=github&utm_medium=repo&utm_campaign=smart_buzzer_n8n) for people who want the same outcome without managing Twilio, n8n, webhooks, and access rules themselves. It's available on iOS and web. 

The DIY workflow in this repository is free to use independently.

## References

- [Twilio speech input](https://www.twilio.com/docs/voice/twiml/gather)
- [Twilio DTMF digits](https://www.twilio.com/docs/voice/twiml/play)
- [Twilio webhook security](https://www.twilio.com/docs/usage/webhooks/webhooks-security)

## License

[MIT](LICENSE)
