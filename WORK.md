# Overview

This doc lists work items believed to be in scope for the multicast community group.

# Docs targeting normative consensus

## W3C

To be authored by this group, targeting adoption by a WG for development as a standard:

 * WebTransport Extension:\
   Formal proposal for a suitable API extension to WebTransport

## IETF

Known high-priority dependencies for this group's scope:

 * [Multicast Security Considerations](https://github.com/squarooticus/draft-krose-multicast-security)
 * [AMBI](https://datatracker.ietf.org/doc/html/draft-ietf-mboned-ambi)

See also:

 * [DORMS](https://datatracker.ietf.org/doc/html/draft-ietf-mboned-dorms) (AMBI dependency)
 * [CBACC](https://datatracker.ietf.org/doc/html/draft-ietf-mboned-cbacc) (may be required for certain threat models)
 * [MNAT](https://datatracker.ietf.org/doc/html/draft-ietf-mboned-mnat) (may be required for certain networks)

Possibly important items for consideration:

 * A new DNS RRType to provide a multicast channel via domain name, instead of IP pair.
 * URL-based generic multicast streams?  (see expired [draft-singer-appsawg-mcast-url](https://datatracker.ietf.org/doc/html/draft-singer-appsawg-mcast-url))
  * perhaps a new url scheme to indicate secured multicast channel?

# Implementation & Test

## API/API components

 * [chromium demo implementation](https://github.com/GrumpyOldTroll/chromium_fork)
   * Needs authentication ([AMBI](https://datatracker.ietf.org/doc/draft-ietf-mboned-ambi/))
   * Probably needs batched payload transfer (or more generally a performance pass)
   * Needs automated tests in the build process
     * with coverage reports
     * with fuzzing
   * May need MNAT support
 * [libmcrx](https://github.com/GrumpyOldTroll/libmcrx) (used by chromium implementation)
   * Needs implementations for:
     * Windows
     * Android
     * IOS
   * Needs packaging for distros
   * Needs test suite with coverage checking
 * Port to more browsers besides Chromium:
   * Firefox at least.  (libmcrx wrapper for rust or separate implementation?)
   * Others as volunteers are willing
 * WPT test suite

## Sample Apps

 * [Demo page](https://htmlpreview.github.io/?https://github.com/GrumpyOldTroll/wicg-multicast-receiver-api/blob/master/demo-multicast-receive-api.html) ([source](https://github.com/GrumpyOldTroll/wicg-multicast-receiver-api/blob/master/demo-multicast-receive-api.html))

# Non-normative Documents

 * [Explainer](https://github.com/GrumpyOldTroll/wicg-multicast-receiver-api/blob/master/explainer.md)

## Tutorials/primers

 * Getting Started:\
   how to receive multicast traffic in a lab web browser, the better to experiment with
 * Building:\
   how to build a browser that can receive multicast
 * Example app:\
   an app that does something useful

## Useful resources

 * Evangelization/enabling docs
   * Case studies
   * ROI/cost analyses
   * Efficiency/savings modeling
 * Notes about devices
   * Gotchas encountered (with model & version, ideally)
   * Confirmed-good reports (with sample configs, ideally)
   * Vendor statements

