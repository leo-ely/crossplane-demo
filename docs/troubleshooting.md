## Troubleshooting

- **GitHub Actions workflow failures**: Check workflow logs for errors (missing Secrets, IAM permissions).
- **Crossplane pods not starting**: Use `kubectl logs -n crossplane-system -l app=crossplane` in a workflow step.
- **AWS provider errors**: Verify IAM permissions and secret credentials in GitHub Secrets.
- **Kyverno policy not enforcing**: Ensure the policy is applied (`kubectl get clusterpolicy`) and check validation
  failure action.
