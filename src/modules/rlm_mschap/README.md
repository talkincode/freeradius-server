# rlm_mschap
## Metadata
<dl>
  <dt>category</dt><dd>authentication</dd>
</dl>

## Summary
Supports MS-CHAP and MS-CHAPv2 authentication. It also enforces the SMB-Account-Ctrl attribute.

## Description

New features were added:
+ LM support to MS-CHAP
+ support  for  SAMBA passwd files. It introduces new files smbpass.c and
  smbpass.h
+ support for MS-CHAPv2. SHA1 digest support was added (sha1.c, sha1.h)
! module is configurable via radiusd.conf and supports instances
! module supports both authorization and authentication. Authorization
  sets authentication to MS-CHAP if any NTLM-related things found.
  During authorization new attributes added to config_items:
	 LM-Password - LM-encoded password
	 NT-Password - NT-encoded password
	 SMB-Account-CTRL - account control flags in SAMBA format
  During  authentication  these  attributes  are  checked  against  data
  provided by NAS.
- RFC 2433 text with MS-CHAPv1 removed. Microsoft attributes are covered
  by RFC 2458, MS-CHAPv2 - RFC 2759. You can obtain them from
  www.rfceditor.org

ZARAZA,
3APA3A@security.nnov.ru

Below is original README by Jay Miller

This is a partial implementation of MS-CHAP for FreeRadius.  The patch
was designed for Cistron-Radius 1.6.4, but the changes to source are
pretty minimal and should work with previous versions.  It is based on
RFC 2433, which is included with this package.

I have tested this successfully using Windows 98 and Windows 2000 with Cisco
AS5300 terminal servers.  I have not tested it in any other environment, so
I can't guarantee it's success everywhere.  I also don't have time to do
much troubleshooting, though I am interested to hear about problems anyone
might have.  If you can fix a problem, then I will incorporate the fix into
the distribution.

Files included:
mschap.c        -   MS-CHAP functions
mschap.h        -   Definitions and prototypes
md4c.c, md4.h   -   RSA Data Security, Inc. MD4 Message-Digest Algorithm
desport.c,
 deskey.c,
 des.h          -   Fast DES by Phil Karn (portable C version)
rfc2433.txt     -   RFC upon which this algorithm is based


ABOUT MS-CHAP

I was driven to write this when a large customer demanded that they be
able to check "Require Encrypted Password" in their Windows Dial-up
Networking.  Testing showed me that, in Windows 2000 at least, this meant
MS-CHAP.  If you want to specify CHAP, then Windows 2000 requires you to
select "Allow unencrypted password". Duh.

MS-CHAP is similar to CHAP.  The NAS creates a challenge string and gives it
to the client.  The client then uses the password to encrypt the challenge
and gives it back to the NAS, who then gives them both to Radius.  Radius
performs the same encryption of the challenge string using the locally stored
password, then compares the result to the response from the client.

The difference between MS-CHAP and CHAP is in the encryption method.  CHAP
performs one MD5 hash to get the response.  MS-CHAP first encrypts the password
with MD4.  It then pads the 16-byte hash out to 21 bytes and divides this
string into 3 parts.  Each 7-byte part is used as a key for a DES encryption
of the challenge string.  The 8-byte results are then concatonated together
into a 24-byte response.

The method just described is called NT-encryption by the RFC.  MS-CHAP is
actually designed for compatability with Microsoft LAN Manager as well.
The response returned by the client actually contains an LM encrypted
response as well as the NT-encrypted password.  This implementation only
uses the NT-encrypted response, which seems to work fine for Windows 98
and Windows 2000.  The RFC also has a number of other specs for allowing the
user to change password and things like that.  None of that has been
implemented here.

A useful extension of this would be in the local storage of passwords.
Theoretically you should be able to store the MD4 hash rather than the
plain text password.  Then the algorithm could pick it up at the next
step and still calculate the result.  The trouble is that MD4 produces a
binary hash.  That is, any values from 0 to 255 is a valid byte, and I
don't know how to store this in a users file.  If it can be done, then
we could add a check attribute called "MS-CHAP-Hash" instead of password
and get both an encrypted protocol and encrypted password storage at the
same time (CHAP requires plain text passwords, while Crypt-Pass requires
an unencrypted protocol).

If you find this useful, please send me a note just so I can feel good
about myself.

Jay Miller
Columbia, MO, US
jaymiller@socket.net
