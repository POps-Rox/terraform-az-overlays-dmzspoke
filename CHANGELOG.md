# v2.0.0

## Breaking Changes
- **Provider Upgrade**: Bumped `azurerm` from `~> 3.116` to `~> 4.20`
- **Terraform Version**: Minimum required Terraform version increased from `>= 1.9` to `>= 1.10`
- **Subnet Network Policies**: `private_endpoint_network_policies_enabled` (bool) is now converted to `private_endpoint_network_policies` (string: "Enabled"/"Disabled") for azurerm 4.x compatibility. Module variable interface preserved for backward compatibility.
- **Dependencies**: This module sources sibling overlays (`resourcegroup`, `azregionslookup`, etc.). Ensure all consumed modules are upgraded to azurerm 4.x-compatible versions to avoid provider constraint conflicts.

## Added
- Added `azapi` provider `~> 2.0` as a required provider for future Azure API integrations

## Changed
- Applied azurerm 3.x → 4.x attribute renames throughout codebase
- Removed `skip_provider_registration` from example provider blocks (replaced by 4.x default behavior)
- Added subscription_id documentation comments to example provider blocks

## Notes
- All `modules/` subdirectories shipped in this repo have been upgraded
- Examples validated with `terraform init -backend=false -upgrade && terraform validate`

# v1.0.0 - <date>

Added
- Add Something you added
