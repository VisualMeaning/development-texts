# Email deliverability

**2024-09-18**

SPF, DKIM, and DMARC are email authentication protocols used to verify the legitimacy of emails, reduce email spoofing, and improve email deliverability.

### SPF
Ensures that the email is being sent from a legitimate source (i.e., an authorized mail server):
* Requirement - an SPF record in DNS is published, listing authorized IP addresses or servers.
* Flow:
  1. When an email is received, the receiving server checks the SPF record for the domain in the Return-Path (envelope sender).
  2. If the sender's IP is listed in the SPF record, the email passes SPF. If not, it fails.

### DKIM
Ensures that the email content hasn’t been tampered with during transit:
* Requirement - a public key is published in the domain's DNS.
* Flow:
  1. The domain owner generates a DKIM signature using a private key and attaches it to the email header.
  2. The receiving server retrieves the public key from DNS and verifies the DKIM signature to ensure the email hasn’t been altered during transit. If the signature matches, the email passes DKIM.

### DMARC
Combines both SPF and DKIM to provide a policy framework for how receiving mail servers should handle unauthenticated emails.
* Requirement - DMARC policy is published in DNS, specifying actions for emails that fail SPF and/or DKIM (e.g., reject, quarantine, or allow).
* Flow:
  1. DMARC checks if the domain in the visible From address matches the domain in the SPF/DKIM authentication.
  2. If both SPF and DKIM pass (or at least one, depending on the policy) and they align with the From domain, the email passes DMARC:
      * for SPF the "Return-Path" should match the "From" domain
      * for DKIM the signature's domain (`d=`) value should match the "From" domain
  3. If not, the receiving server follows the action specified in the DMARC policy (e.g., mark as spam, reject).

### Sending through Amazon SES
Currently we are using Amazon SES ([authentication docs](https://docs.aws.amazon.com/ses/latest/dg/email-authentication-methods.html)) to send access emails on behalf of "@visual-meaning.com".
It by default attaches a DKIM signature with a published public key managed by the service. As long as we [publish the SPF records](https://docs.aws.amazon.com/ses/latest/dg/send-email-authentication-spf.html), emails should be sent with DKIM and SPF verified.

These defaults do offer verified SPF and DKIM, email are still sent from amazons server and DMARC domain alignment is not possible.
To send an email you need to create and verify an identity, that is of two types: an individual email address or domain (subdomain) identity that covers all mail using it. Note that if configured for example.com, you cannot send from some@sub.example.com. The identity also provides customizations to ensure DMARC compliance.

Compliance through SPF is done [using a custom MAIL FROM domain](https://docs.aws.amazon.com/ses/latest/dg/mail-from.html). This can be configured in both email address and domain indentity.

Compliance through DKIM [setting custom DKIM keys](https://docs.aws.amazon.com/ses/latest/dg/send-email-authentication-dmarc.html#send-email-authentication-dmarc-dkim). In Amazon SES you can only do so in a domain identity.
It is considered a good practise to configure a subdomain instead of configuring for the base domain. Some benefits would be:
- configurational isolation
- component segmentation - gives a context and separate the types of emails
- branding risk - when using third party emailing providers we can setup multiple subdomain so that it does not affect a single one.

**Note:** If custom DKIM keys are configured, Amazon SES will attach it in addition to the default one. In the case of multiple DKIM signatures, the [specification](https://datatracker.ietf.org/doc/html/rfc7489#section-3.1.1) states that one DKIM domain alignment is enough. The reality is email servers have different algorithm to check DKIM, some check both, some [check only one](https://techcommunity.microsoft.com/t5/exchange/incorrect-processing-of-messages-with-multiple-dkim-signatures/td-p/4053047) at random and try to align domain with it. This does not guarantee DMARC compliance through DKIM.