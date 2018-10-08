LIVE GRAPHIC & BROWSER SOURCE TEST SUITE #4
====================

[Related user documentation]()
> Block quotes denote expected outcomes to confirm

---
**These steps validate the following features:**
- Campaign managers can invite, activate, and deactivate broadcasters on a campaigns
- Broadcasters can accept or decline campaign invitations
- Live graphics in a campaign are displayed in a broadcaster's browser source
- Campaign managers can edit live graphic duration times
---
**Prerequisites:**
- A team manager account with two active campaigns, the second of which has live graphic intervals enabled
- An existing broadcaster account with no campaigns
- Various campaign assets
---
**Steps to perform:**

**From a Campaign Manager Account**

1. Invite a broadcaster to a campaign that has at least 2 live graphics
2. Invite a broadcaster to second campaign that has live graphic intervals enabled
> The broadcaster receives an invitation email, and the "Sent Invites" modal shows the invitation

**From a Broadcaster Account**

3. Under the broadcaster dashboard, decline the first campaign invitation and accept the second invitation
> Broadcaster actions related to that campaign are created and shown in the action queue

**From a Campaign Manager Account**

4. Activate the broadcaster on the first campaign
> Live graphics now appear on the broadcaster's browser source alternating with the default duration time, and he is moved from the Inactive column to the Active column

5. Re-invite the broadcaster to the second campaign that has at least 2 live graphics
6. Label one live graphic component without labeling the broadcaster accordingly
> The live graphic is removed instantly from the browser source queue, without the need to refresh

7. Remove the label on the live graphic
> The live graphic is added instantly to the browser source queue, without the need to refresh

8. Label one live graphic component and label the broadcaster accordingly
> The live graphic remains in the browser source queue, without the need to refresh

9. Delete one live graphic component
> The live graphic is removed instantly from the browser source queue, without the need to refresh

**From a Broadcaster Account**

10. Under the broadcaster dashboard, accept the second campaign invitation

**From a Campaign Manager Account**

11. Activate the broadcaster on the second campaign
> Live graphics from both campaigns now appear on the broadcaster's browser source

12. Create a live graphic component with a specific duration time (Accepted file types JPG, JPEG, PNG, GIF, WEBM, MP4)
> The live graphic is added instantly to the browser source queue, without the need to refresh, and is shown on screen for the specified duration time

13. Edit the live graphic component image and duration
> The new live graphic is replaced instantly in the browser source queue, without the need to refresh, and is shown on screen for the specified duration time

14. Set the broadcaster as inactive for the second campaign
> Live graphics from that campaign are removed instantly from the browser source queue, and the broadcaster is moved from the Active column to the Inactive column

15. On a new tab, open the broadcaster's browser source again
> Nothing happens on the first browser source; A new browser source instance being loaded should not affect the live graphic rotation of another
