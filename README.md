# Mastodon Helm Chart
<a href="https://github.com/small-hack/mastodon-helm-chart/releases"><img src="https://img.shields.io/github/v/release/small-hack/mastodon-helm-chart?style=plastic&labelColor=blue&color=green&logo=GitHub&logoColor=white"></a>

[small-hack/mastodon-helm-chart](https://github.com/small-hack/mastodon-helm-chart) is a fork of the official mastodon helm chart for installing Mastodon on a Kubernetes cluster. We'll maintain this at least till some of the security features PRs are merged in the upstream repo. The basic usage is:

```bash
# add the chart repo to your helm repos
helm repo add mastodon https://small-hack.github.io/mastodon-helm-chart

# download the values.yaml and edit it with your own values such as YOUR hostname
helm show values mastodon/mastodon > values.yaml

# install the chart
helm install --namespace mastodon --create-namespace mastodon/mastodon --values values.yaml
```

This chart is tested with k8s 1.27+ and helm 3.6.0+.

> [!Note]
> We may publicly archive this repo in the near future if [bitnami/charts#19179](https://github.com/bitnami/charts/pull/19179) is merged and the chart works. Feel free to take what you need though :)

## Known caveats for this chart
Currently, you need to run PostgreSQL and Redis helm charts independently of this one, because there's a helm hook job called db-migrate that we can't figure out how to make run after the dependency charts are fully installed, but before everything else. If you know the answer to this, please open an issue/pr here and let us know!

# Configuration

The variables that _must_ be configured are:

- password and keys in the `mastodon.secrets`, `postgresql`, and `redis` groups; if
  left blank, some of those values will be autogenerated, but will not persist
  across upgrades.

- SMTP settings for your mailer in the `mastodon.smtp` group.

If your PersistentVolumeClaim is `ReadWriteOnce` and you're unable to use a S3-compatible service or
run a self-hosted compatible service like [Minio](https://min.io/docs/minio/kubernetes/upstream/index.html)
then you need to set the pod affinity so the web and sidekiq pods are scheduled to the same node.

Example configuration:
```yaml
podAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
          - key: app.kubernetes.io/part-of
            operator: In
            values:
              - rails
      topologyKey: kubernetes.io/hostname
```

# Administration

You can run [admin CLI](https://docs.joinmastodon.org/admin/tootctl/) commands in the web deployment.

```bash
kubectl -n mastodon exec -it deployment/mastodon-web -- bash
tootctl accounts modify admin --reset-password
```

or
```bash
kubectl -n mastodon exec -it deployment/mastodon-web -- tootctl accounts modify admin --reset-password
```

# Missing features

Currently this chart does _not_ support:

- Hidden services
- Swift

# Upgrading

Because database migrations are managed as a Job separate from the Rails and Sidekiq deployments, it’s possible they will occur in the wrong order. After upgrading Mastodon versions, it may sometimes be necessary to manually delete the Rails and Sidekiq pods so that they are recreated against the latest migration. If you're upgrading from a version before 3.x to a version before 4.x, please see the upstream mastodon chart as that is before our fork.
