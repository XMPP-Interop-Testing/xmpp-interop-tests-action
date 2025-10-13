# xmpp-interop-tests-action
A GitHub Action that performs XMPP interoperability tests on an XMPP domain.

For more information, please visit our project website at https://xmpp-interop-testing.github.io/

## Inputs
The Action can be configured using the following inputs:
- `host`: IP address or DNS name of the XMPP service to run the tests on. Default value: `127.0.0.1`
- `domain`: the XMPP domain name of server under test. Default value: `example.org`
- `timeout`: the amount of milliseconds after which an XMPP action (typically an IQ request) is considered timed out. Default value: `5000` (five seconds)
- `adminAccountUsername`: _(optional)_ The account name of a pre-existing user that is allowed to create other users, per [XEP-0133](https://xmpp.org/extensions/xep-0133.html). See: "[Provisioning Test Accounts](#provisioning-test-accounts)"
- `adminAccountPassword`: _(optional)_ The password of the admin account.
- `accountOneUsername`: _(optional)_ The first account name of a set of three accounts used for testing. See: "[Provisioning Test Accounts](#provisioning-test-accounts)"
- `accountOnePassword`: _(optional)_ The password of the accountOneUsername account.
- `accountTwoUsername`: _(optional)_ The second account name of a set of three accounts used for testing. See: "[Provisioning Test Accounts](#provisioning-test-accounts)"
- `accountTwoPassword`: _(optional)_ The password of the accountTwoUsername account
- `accountThreeUsername`: _(optional)_ The third account name of a set of three accounts used for testing. See: "[Provisioning Test Accounts](#provisioning-test-accounts)"
- `accountThreePassword`: _(optional)_ The password of the accountThreeUsername account.
- `disabledTests`: _(optional)_ A comma-separated list of tests that are to be skipped. For example: `EntityCapsTest,SoftwareInfoIntegrationTest`
- `disabledSpecifications`: _(optional)_ A comma-separated list of specifications (not case-sensitive) that are to be skipped. For example: `XEP-0045,XEP-0060`
- `enabledTests`: _(optional)_ A comma-separated list of tests that are the only ones to be run. For example: `EntityCapsTest,SoftwareInfoIntegrationTest`
- `enabledSpecifications`: _(optional)_ A comma-separated list of specifications (not case-sensitive) that are the only ones to be run. For example: `XEP-0045,XEP-0060`
- `logDir`: _(optional)_: The directory in which the test output and logs are to be stored. This directory will be created, if it does not already exist. Default value: `./output`
- `failOnImpossibleTest`: _(optional)_ If set to 'true', fails the test run if any configured tests were impossible to execute. Default value: `false`
- `testId`: _(optional)_: Used to differentiate generated artifacts. Only useful when using this Action more than once in the same workflow. Default value: `default`

### Provisioning Test Accounts

To be able to run the tests, the server that is being tested needs to be provisioned with test accounts. Three different mechanisms can be used for this:
- **Admin Account** - By configuring the username and password of a pre-existing administrative user, using the `adminAccountUsername` and `adminAccountPassword` configuration options, three test accounts will be created using [XEP-0133: Service Administration](https://xmpp.org/extensions/xep-0133.html) functionality.
- **Explicit Test Accounts** - You can configure three pre-existing accounts that will be used for testing, using the `accountOneUsername`, `accountOnePassword`, `accountTwoUsername`, `accountTwoPassword`, `accountThreeUsername` and `accountThreePassword` configuration options.
- **In-Band Registration** - If no admin account and no explicit tests accounts are provided, in-band registration ([XEP-0077](https://xmpp.org/extensions/xep-0077.html)) will be used to provision accounts.

## Basic Configuration
```yaml
uses: XMPP-Interop-Testing/xmpp-interop-tests-action@v1.7.2
with:
  domain: 'shakespeare.lit'
  adminAccountUsername: 'juliet'
  adminAccountPassword: 'O_Romeo_Romeo!'
```

## Usage Example
It is expected that this action is used in a continuous integration flow that creates a new build of the XMPP server
that is to be the subject of the tests.

Very generically, the xmpp-interop-test-action is expected to be part of such a flow in this manner:
1. Compile and build server software
2. Start server
3. **Invoke xmpp-interop-test-action**
4. Stop server

This could look something like the flow below. Note that all but the third step in this flow are placeholders. They need to be replaced by steps that are specific to the server that is being build and tested. 

```yaml
- name: Download Server distribution artifact from build job.
  uses: actions/download-artifact@v4
  with:
    name: my-server-distribution
    path: .

- name: Start CI server from distribution
  id: startCIServer
  uses: ./.github/actions/startserver-action # Should result in a running server.

- name: Run XMPP Interoperability Tests against CI server.
  uses: XMPP-Interop-Testing/xmpp-interop-tests-action@v1.7.2
  with:
    domain: 'shakespeare.lit'
    adminAccountUsername: 'juliet'
    adminAccountPassword: 'O_Romeo_Romeo!'

- name: Stop CI server
  if: ${{ always() && steps.startCIServer.conclusion == 'success' }}
  uses: ./.github/actions/stopserver-action
```
