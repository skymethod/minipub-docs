---
title: Funding Minipub
order: 3
---

# Funding Minipub
This is an independent open-source project by [John Spurlock](https://github.com/johnspurlock-skymethod) with no financial backing, just an interest in helping open podcasting comments get off the ground.

If you support the goals of this project and want to help fund its ongoing development, you can send a donation our way using the button below, thanks!

<Button type="primary" href="https://buy.stripe.com/6oE2aIduL5rzfEQfYZ">Donate →</Button>

### Micropayments
If you wish to send streaming micropayments to Minipub over the Lightning network, add the following info to your RSS feed's [value tag](https://github.com/Podcastindex-org/podcast-namespace/blob/main/docs/1.0.md#value):

```xml
<podcast:value type="lightning" method="keysend">
   ...
   <podcast:valueRecipient name="Minipub" address="02c85c34b89d4f5b16f60fe6f524862669ddc34f8bb7f8ce9ec783c1db6c8da857" type="node" customKey="7629171" customValue="minipub" fee="true" split="5"/>
</podcast:value>
```
