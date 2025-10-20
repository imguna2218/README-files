**This is the only change you need to make. It is a system change, not a code change.**

1.  **Open the `sudoers` file for editing.** This MUST be done with the `visudo` command, which prevents you from saving a broken file that could lock you out of your system.

    ```bash
    sudo visudo
    ```

    --

    ```bash
    which chown
    ```
    with the path obtained .... ex : `/usr/bin/chown`

3.  **Add the following line to the end of the file.** Scroll all the way to the bottom and add this exact line. Replace `gunasekhar` with your actual username if it's different.

    ```
    gunasekhar ALL=(ALL) NOPASSWD: /usr/local/bin/isolate, /usr/bin/chown
    gunasekhar ALL=(ALL) NOPASSWD: /home/gunasekhar/evalx/target/release/evalx worker
    ```

      * **`gunasekhar`**: The user who gets the permission.
      * **`ALL=(ALL)`**: On any machine, acting as any user.
      * **`NOPASSWD:`**: Do not ask for a password. This is critical for an automated service.
      * **`/usr/local/bin/isolate`**: The *only* command this rule applies to. (Verify this is the correct path to your `isolate` binary by running `which isolate`).

4.  **Save and exit the editor.** In `nano` (the default for `visudo` on many systems), this is `Ctrl+X`, then `Y`, then `Enter`.

5.  After that
   ```bash
        
