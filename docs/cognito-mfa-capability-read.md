# Reading the Cognito pool MFA capability

The Cognito template grants its managed `AuthenticatedRole` and `RegisteredRole` the single action `cognito-idp:GetUserPoolMfaConfig`, scoped to the ARN of the user pool created by that template. Existing role trust, API permissions, group precedence and Identity Pool role selection are preserved. No additional backend or runtime artifact is required.

Both roles matter: the Identity Pool's default authenticated role is used when no role mapping or explicit custom role is selected; the registered and admin groups share the registered role. The template does not change that selection behavior.

Applications can use the current authenticated Identity Pool session's temporary credentials to make a SigV4-signed Cognito request. `WebAuthnConfiguration.FactorConfiguration` describes the pool capability. This is separate from a user's WebAuthn MFA opt-in and does not authorize changing that preference.

The API returns the full pool MFA configuration, including relying-party ID, factor settings and applicable message/SMS configuration references. It is not a filtered boolean-only endpoint. Deployments requiring a narrower response need their own adapter. Credentials must not be copied into public configuration.

## Existing or externally managed resources

The template creates these roles only when the template, new user pool and new Identity Pool are all enabled. Providing `ExistingAuthenticatedRoleArn` or `ExistingRegisteredRoleArn` exposes those references; it does **not** authorize this template to edit the external roles.

The owner of externally managed roles must attach the following policy to each role actually used by signed-in identities. Replace the example resource with the exact existing pool ARN in its real partition, account and region. Do not use a wildcard pool ARN.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["cognito-idp:GetUserPoolMfaConfig"],
      "Resource": "arn:aws:cognito-idp:eu-central-1:111122223333:userpool/eu-central-1_Example"
    }
  ]
}
```

Check permission boundaries and organizational controls as well. If an application has no Identity Pool credentials or lacks this permission, it should report the capability as unknown and preserve its existing sign-in and MFA behavior. Successful operator access or an IAM simulation alone does not prove an authenticated browser request.

This permission change does not enable or disable TOTP, email/SMS MFA, WebAuthn MFA, recovery, device tracking or any user's opt-in.
