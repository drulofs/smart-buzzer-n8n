# Setup Guide

This guide connects a Twilio phone number to the n8n magic word buzzer workflow.

```text
Building buzzer
  -> Twilio phone number
  -> n8n webhook
  -> magic word check
  -> TwiML response
  -> DTMF unlock tone
```

The workflow is a reference implementation. Review the limitations in the main README before connecting it to a real intercom.

## Step 1: Configure a Twilio number

In Twilio, buy or configure a phone number with Voice capability. This is the number you will eventually assign to your building buzzer.

Some buildings may reject VoIP numbers or may not reliably accept DTMF tones from Twilio. Test before relying on it.

## Step 2: Import the workflow

In n8n:

1. Create a new workflow.
2. Import `workflows/smart-buzzer-n8n.json`.
3. Open **Incoming Intercom Call**.
4. Copy the **Production URL**, not the test URL.
5. Leave the workflow inactive while you configure it.

The production webhook must be publicly reachable over HTTPS so Twilio can call it.

## Step 3: Set the magic word and unlock digit

Open the **Configuration** node and replace:

```text
magicWord: open sesame
unlockDigit: 6
```

Use the digit your building normally expects. Choose a magic word that is easy to say but difficult to guess accidentally. Do not reuse an important password.

The public example uses a case-insensitive `contains` comparison so punctuation and surrounding words do not prevent a match.

## Step 4: Configure the Twilio Voice webhook

In Twilio:

1. Go to **Phone Numbers**.
2. Select the number you want to use.
3. Open its Voice configuration.
4. Set the handler for incoming calls to **Webhook**.
5. Paste the n8n production webhook URL.
6. Set the HTTP method to `POST`.
7. Save the configuration.

When someone calls the Twilio number, Twilio sends the call data to n8n. On the first request, n8n returns a speech `<Gather>` prompt. Twilio then posts the transcription back to the same workflow as `SpeechResult`.

## Step 5: Activate and test the magic word flow

Activate the workflow, then call the Twilio number from a normal phone.

Test at least three cases:

1. Say the configured magic word and confirm you hear “Access granted.”
2. Say an incorrect magic word and confirm the call is rejected.
3. Say nothing and confirm the call ends without sending the unlock digit.

Check the n8n execution data and confirm the follow-up request contains `body.SpeechResult`.

Calling from a normal phone verifies the Twilio, transcription, and n8n path. It does not prove that the building intercom will accept the number or recognize the DTMF tone.

## Step 6: Connect the building intercom

Ask your building manager, concierge, or intercom administrator to assign the Twilio number to your unit.

Then test the complete flow from the building entrance:

1. Call the unit from the intercom.
2. Say the correct magic word.
3. Confirm the door unlocks.
4. Repeat with an incorrect magic word and confirm it stays locked.
5. Confirm the call closes cleanly when nobody speaks.

If the door does not unlock, adjust the unlock digit or DTMF delay and review [`troubleshooting.md`](troubleshooting.md).

## Step 7: Protect the webhook

This workflow does not validate that requests came from Twilio.

Twilio signs webhook requests in the `X-Twilio-Signature` header. Before real use, validate that signature using the exact webhook URL and request parameters. A common pattern is to validate requests in a reverse proxy, small application, or Twilio Function before forwarding them to n8n.

Do not put a Twilio Auth Token directly into a workflow you plan to export or publish. See [Twilio's webhook security documentation](https://www.twilio.com/docs/usage/webhooks/webhooks-security).

## Step 8: Add stronger access rules

For real use, consider combining the magic word with:

- approved access windows
- booking or calendar checks
- one-time or rotating magic words
- failed-attempt limits
- logs and notifications
- a quick disable switch
- a manual fallback

When a dependency fails, deny access rather than silently unlocking.
