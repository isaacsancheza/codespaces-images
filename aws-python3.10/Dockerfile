FROM ghcr.io/isaacsancheza/images:python3.10

# install pydot, cfn-lint, and cookiecutter
RUN pip --disable-pip-version-check --no-cache-dir install pydot pyyaml chalice cfn-lint aws-cdk-lib chalice[cdkv2] cookiecutter cfn-lint-serverless
