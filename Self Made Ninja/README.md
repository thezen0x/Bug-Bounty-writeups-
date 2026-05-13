Title:
2FA Code Can Be Reset by Page Refresh After VPN/IP Change – Weak Session Binding 
Severity:
Medium (or Low-Medium depending on your company’s policy)
Summary:
The 2FA verification process does not properly bind the challenge to the original session and source IP. An attacker (or user) with valid credentials can bypass the initial VPN/IP enforcement by refreshing the page after the first code expires, receiving a fresh 2FA code without re-authenticating from the correct VPN.
Steps to Reproduce (STR):
    1. Disconnect from company VPN (or use any non-corporate IP).
    2. Go to the login portal and enter valid Username + Password.
    3. On the 2FA page, wait for the code to be generated (timer starts).
    4. Enter the 2FA code while not connected to VPN → code is rejected (as expected).
    5. Switch to company VPN.
    6. Refresh the 2FA page (F5 or browser refresh).
    7. A new 2FA code is immediately displayed with a fresh timer.
    8. Enter the new code → login succeeds.
Expected Behavior:
    • The 2FA challenge must be tightly bound to the original authentication attempt + source IP/session token.
    • Refreshing the page after rejection or timeout should either:
        ◦ Show the same expired code, or
        ◦ Immediately invalidate the session and force a full re-login from scratch.
    • Once a code is rejected due to VPN policy, no new code should be issued on the same session without re-entering credentials.
Actual Behavior:
Refreshing the page after VPN switch generates a completely new valid 2FA code and allows successful login.
Impact:
    • Allows bypassing strict “VPN-only” 2FA enforcement.
    • An attacker with stolen username + password can use this trick to log in from outside the corporate network.
    • Reduces effectiveness of the time-limited 2FA and IP-based controls.
    • In a real attack this could lead to unauthorized access to corporate systems/data.
Environment:
    • Browser: Brave
    • 2FA Method: Proton Pass
    • VPN: Proton VPN,  WireGuard  
    • Affected URL: https://git.selfmade.ninja/users/sign_in
Evidence:


Suggested Fix:
    1. Bind the 2FA challenge to both the session token and the source IP (or a hashed VPN identifier) at the moment the challenge is issued.
    2. On page refresh or new GET request to the 2FA endpoint, re-validate the original IP/session and reject if changed.
    3. Implement proper “single-use + timeout” state on the backend instead of stateless refresh.
    4. Add rate limiting on 2FA challenge generation per session/IP.
Additional Notes:
This issue was discovered accidentally during normal login. No malicious intent.
