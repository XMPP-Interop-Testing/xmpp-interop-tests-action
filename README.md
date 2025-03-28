# xmpp-interop-tests-action
A GitHub Action that performs XMPP interoperability tests on an XMPP domain.

For more information, please visit our project website at https://xmpp-interop-testing.github.io/

## Test Account Provisioning
The Action will typically execute various tests. Each test will use a fresh set of XMPP user accounts. These are
automatically provisioned by the testing framework. They will be removed after the test execution.

The following strategies for test account provisioning are supported:
- By default, the test accounts are provisioned using XMPP's "In-band Registration" functionality (as defined in 
  [XEP-0077](https://xmpp.org/extensions/xep-0077.html)).
- Alternatively, test accounts can be provisioned using XMPP 'Ad-hoc commands', as specified in 
  [XEP-0133: Service Administration](https://xmpp.org/extensions/xep-0133.html). To enable this way of provisioning, the
  Action's configuration must include the optional `adminAccountUsername` and `adminAccountPassword` inputs (as 
  documented below).

## Inputs
The Action can be configured using the following inputs:
- `host`: IP address or DNS name of the XMPP service to run the tests on. Default value: `127.0.0.1`
- `domain`: the XMPP domain name of server under test. Default value: `example.org`
- `timeout`: the amount of milliseconds after which an XMPP action (typically an IQ request) is considered timed out. Default value: `5000` (five seconds)
- `adminAccountUsername`: _(optional)_ The account name of a pre-existing user that is allowed to create other users, per [XEP-0133](https://xmpp.org/extensions/xep-0133.html). If not provided, in-band registration ([XEP-0077](https://xmpp.org/extensions/xep-0077.html)) will be used to provision accounts.
- `adminAccountPassword`: _(optional)_ The password of the admin account.
- `disabledTests`: _(optional)_ A comma-separated list of tests that are to be skipped. For example: `EntityCapsTest,SoftwareInfoIntegrationTest`
- `disabledSpecifications`: _(optional)_ A comma-separated list of specifications (not case-sensitive) that are to be skipped. For example: `XEP-0045,XEP-0060`
- `logDir`: _(optional)_ The directory in which the test output and logs are to be stored. This directory will be created, if it does not already exist. Default value: `./output`

## Basic Configuration
```yaml
uses: XMPP-Interop-Testing/xmpp-interop-tests-action@v1.5.0
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
  uses: XMPP-Interop-Testing/xmpp-interop-tests-action@v1.5.0
  with:
    domain: 'shakespeare.lit'
    adminAccountUsername: 'juliet'
    adminAccountPassword: 'O_Romeo_Romeo!'

- name: Stop CI server
  if: ${{ always() && steps.startCIServer.conclusion == 'success' }}
  uses: ./.github/actions/stopserver-action
```
