# JIT diagnostics on iOS 26 and later

Henge needs executable JIT memory, not merely a debugger flag. A successful
`Script Execution Completed` message is **not** confirmation that JIT works.
Check Henge's JIT log for `JIT region secured` (or a successful region
self-test) before installing macOS.

## `attach_response = E96` then `Socket NotConnected`

The debugserver rejected `vAttach`, so the socket cannot safely be used.
In older versions of `henge-jit.js`, the script would still send `c`, producing
the secondary `NotConnected` error. The corrected script stops immediately
and logs `JIT_ATTACH_FAILED` instead. It cannot override a genuine attach
refusal by iOS.

Things to check on the device:

1. Enable Developer Mode, then check that LocalDevVPN is connected and no
   other VPN intercepts the local route.
2. Refresh the device pairing file in StikDebug or in Henge's built-in
   pairing setup. A stale or incompatible pairing can prevent a debugger
   attach; keep pairing files private.
3. In the built-in flow, tap **Check setup** to prepare the Developer Disk
   Image (DDI), then **Enable JIT**. Use **Reset Developer Disk Image**
   only when DDI preparation itself fails.
4. Launch Henge normally and choose **Turn On JIT** from Henge. Merely
   opening its entry inside StikDebug may attach without the Henge JIT script.
5. Review the detailed StikDebug/LocalDevVPN log if `E96` persists; its
   exact cause cannot be determined from the two-character reply alone.

## `Henge's JIT helper is missing from this installation`

The built-in flow requires **both** files inside the *installed* app:

- `PlugIns/HengeJITHelper.appex/Info.plist`
- `PlugIns/HengeJITHelper.appex/HengeJITHelper`

The build script now verifies the helper's signature and these files in the
exported IPA. However, an external sideloader/resigner can still strip or
fail to install the extension. When signing with KSign or another tool,
preserve the `.appex` and sign the extension as well as the parent app.
If the signing service cannot preserve extensions, choose **StikDebug**
instead of **Built-in StikJIT**.

The app now detects incomplete helper bundles and avoids choosing them in
automatic mode when StikDebug is available.

## Verification levels

- **Offline test pass:** only proves that the script handles simulated
  debugserver replies.
- **IPA bundle check pass:** proves the extension was present in the output
  archive before installing, not that the sideloader kept it.
- **Real device pass:** requires checking the installed extension, a
  successful debugger attach and an executable JIT-region self-test on iOS.

Do not share device pairing files in bug reports.
