---

tags:
  - Zentyal

---

# Bugfixing

On this page, we will briefly explain and propose solutions to the bugs found as of (2023-02) that affect the deployment of Zentyal in the project we have created.

## Webadmin

Below are the bugs found in the Webadmin module.

### Saving changes

In version `7.0.5` of the `zentyal-core` module, there is [this bug] that causes the administration interface to always display the message "saving changes".

[this bug]: https://github.com/zentyal/zentyal/issues/2100

To temporarily solve the issue, do the following:

1. In the file `/usr/share/perl5/EBox/WebAdmin.pm`, modify the _daemons method of line 99 with the following content:

    ```perl
    sub _daemons
    {
        return [
            { name => 'zentyal.webadmin-uwsgi' },
            { name => 'zentyal.webadmin-nginx' }
        ];
    }
    ```

2. Restart the Webadmin module:

    ```sh linenums="1"
    sudo zs webadmin restart
    ```

With this, the bug will be temporarily fixed.

## Webmail

Below are the bugs found in the Webmail module.

### IMAPS

The configuration established by Zentyal for the IMAPS protocol in the Webmail module is correct as long as a recognized certificate is used. If the default certificate is used, an error will be displayed when logging in, indicating that it cannot access the user's mailbox.

This bug is present in version `7.0.0` of the `zentyal-sogo` package. We can see the version of our package by running the command: `sudo dpkg -l zentyal-sogo`.

There are several solutions to this problem:

1. We can modify the configuration parameter to allow insecure certificate connections on localhost.
2. We can temporarily enable the `IMAP` protocol from `Mail -> General`.
3. We can use a recognized certificate in the Webmail (Sogo) module, as explained on the [Certificates](https://zentyal-aws.projects.djoven.es/en/zentyal-certificates/) page.

If we want to apply the first option, we need to perform the following actions:

1. In the file `/usr/share/perl5/EBox/SOGo.pm`, edit line **265** and set the following content:

    ```sh linenums="1"
    my $imapServer = ($mail->imap() ? '127.0.0.1:143' : '"imaps://127.0.0.1:993/?tlsVerifyMode=allowInsecureLocalhost"');
    ```

2. Restart the modules: Webmail and Sogo:

    ```sh linenums="1"
    sudo zs webadmin restart
    sudo zs sogo restart
    ```

3. Try to log in to Webmail again to confirm the solution.
