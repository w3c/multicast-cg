
# Proposed Agenda

 1. Welcome, Agenda-bash    (~5 mins)
 2. Intro to Multicast-CG (15 mins)
 3. Next Steps: discuss briefly & schedule working sessions (25 mins)
    1. QUIC with multicast (https://datatracker.ietf.org/doc/html/draft-pardue-quic-http-mcast)
       1. Get basic functionality demo running (no browser) with <https://github.com/bbc/nghq>
       2. Also look into the latest [webtransport implementation](https://github.com/web-platform-tests/wpt/tree/master/tools/webtransport/h3), which has datagrams. See about hacking in multicast like nghq did?
       3. Once something works, get it running in browser.
          1. datagrams+webtransport better match to existing api?  nghq's object approach better match for xhr, I think.  Any others?
       4. Look into addding AMBI to quic-http-mcast (<https://datatracker.ietf.org/doc/html/draft-ietf-mboned-ambi>)
    2. Anyone know anyone willing who hasn't chimed in on IETEF secdispatch support for work on <https://datatracker.ietf.org/doc/html/draft-krose-multicast-security>?
    3. IETF hackathon to play VLC mcast streams from internet2 in browser demo.  Thread: <https://lists.geant.org/sympa/arc/multicast-discuss/2021-10/msg00000.html>
       1. This can anchor the datagrams use case and carry forward if we can get it working.
 4. Wrap-up & Summary (5 mins)

# Logistics

Please note:

 1. This meeting will be recorded if nobody withholds consent, as covered by the latest draft process guidelines:\
<https://www.w3.org/Consortium/Process/Drafts/#meeting-recording>

  The purpose of recording is to aid with minutes, enable later review of comments, and to help future new or prospective group members get oriented.  The video will be posted to Youtube and made publicly available without an expiration.  However, if requested by recorded participants, sections of the public recording will be redacted to their satisfaction as needed, before or after publication (though after publication of course there may be copies outside our control).  A link to the recording will be kept in the meeting log here (and updated if replaced with a redacted recording):
  <https://github.com/w3c/multicast-cg/tree/main/meetings>

 2. This is a W3C Community Group meeting, so it's to operate under
the W3C Code of Ethics and Professional Conduct:
  <https://www.w3.org/Consortium/cepc/>

# Coordinates

 1. Meeting link should work for most (webex client install if missing):\
   <https://akamai.webex.com/akamai/j.php?MTID=m82facd432d2daa720ebbb302d17e256c>

  Otherwise, <https://akamai.webex.com/akamai> should let you put in the meeting number and password:

   * Meeting number: 2598 001 6648
   * Password: yay-multicast!

 2. Join by phone:
  
   * 1-408-792-6300 Call-in number (US/Canada)
   * 1-877-668-4490 Call-in toll-free number (US/Canada)

 3. Join by video system:

   * Dial 25980016648@akamai.webex.com
   * Access code: 2598 001 6648

Global call-in numbers: <https://akamai.webex.com/webappng/sites/akamai/meeting/info/ace9a972e8c04a7e8c01ea2f20397734#>

Toll-free calling restrictions: <https://www.webex.com/pdf/tollfree_restrictions.pdf>


