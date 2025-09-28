---
title: Migrate Your Existing Matrix Account, The Harder but Safe Way!
date: 2024-10-08
authors:
  - name: SoupGlasses
    link: https://finnes.dev
    image: https://github.com/soupglasses.png
---

I love migrating my federated accounts on Mastodon, i have done so multiple times in my years using it. Its painless, its easy, and everything is done for me. Its like magic! So today I decided to also migrate my federated Matrix away from matrix.org, should be just as easy right?

From a quick search online, there exists a preexisting solution called [EMS Matrix Migration](https://ems.element.io/tools/matrix-migration) which should work similar to the migration of a mastodon account. But, how they achieve this is by logging into your previous and new account and then running a bunch of commands on your behalf. I do not particularly trust this method, since this would be giving away your password to a third party for a secure messenger protocol! How this is the “official way” is just baffling me. Also there not being an integrated way to do this in Element either, its only available in the paid and proprietary EMS toolbox. So to hell with this solution, lets just do it ourselves. How hard could that be?

## The Migration Strategy

1. Ensure you have registered your new account and have it set up with E2EE keys. This should be done automatically as long as you have logged into the account at least once.
2. Log into your old account using Element, and find the list of direct message rooms you want to migrate. For each of these rooms, write the following commands. This will invite your new account, and set them to an admin when you join with the new account.

  ```
  /invite @new_account:example.com Account Migration/op @new_account:example.com 100
  ```

3. Export your “All Settings” —> “Security and Privacy” —> “Export E2E room keys”. Give it a password you can remember and download the file.

4. Now log into your new account using Element, and go to the “All Settings” —> “Security and Privacy” —> “Import E2E room keys“ and type in the password you gave the file earlier.

5. Once this has been imported, you can start joining the rooms you have been invited to. A handy tip is to go to “All Settings” —> “Security and Privacy” —> “Accept all N invites“ button to let your client do this for you.

6. When all the rooms are joined, you may use the `/part` command  on your old account to leave the rooms. Or alternatively kick your old account from the new account using the following command.

  ```
  /kick @old_account:example.com Account Migration
  ```

## Relabeling Rooms back to Direct Messages

You may notice all the direct message rooms are seen as “Group Rooms”. This is an annoying side effect of joining rooms manually after they were created. However, there is a solution. You can manually label rooms as direct rooms as seen in [this GitHub issue](https://github.com/element-hq/element-web/issues/15474). This is a bit labor intensive sadly, but the strategy goes as follows.

1. In any room, type `/devtools` and open the "Explore Account Data" option (and not the "Export Room Data" button).

2. If `m.direct` exists, click on it. If it doesn't, click the button at the bottom named "Send custom account data event" and after that set the event type to `m.direct`.

3. In the data field, you will need to write out some JSON. So be careful with your commas and brackets. If yours is empty it will follow a pattern like the following, repeat as many times as needed:

  ```json
  {
    "@account_one:example.com": [
      "!room_id:example.com"
    ],
    "@account_n:example.com": [
      "!room_id:example.com"
    ]
  }
  ```

4. You need to find the matrix handle, you can find through the Information button at the top right when viewing a room, clicking Profiles, and choosing the person. And also while you are in this room, you can further write `/devtools` and find the internal room id that you need to add into the array at the top of the menu that pops up.

5. Once you're sure the JSON is correct, click "Send." The rooms will automatically be marked as direct messages and start fetching the profile pictures and renaming the room for you.

That's it! Add your actual group rooms by copying the invite link or invite them as normal from your old account. Then you should have successfully migrated your matrix account with a lot of effort.

Happy chatting!
