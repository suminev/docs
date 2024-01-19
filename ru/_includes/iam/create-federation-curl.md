Чтобы создать федерацию, воспользуйтесь методом REST API [create](../../organization/api-ref/Federation/create.md) для ресурса [Federation](../../organization/api-ref/Federation/index.md) или вызовом gRPC API [FederationService/Create](../../organization/api-ref/grpc/federation_service#Create) и передайте в запросе файл с параметрами запроса.

Пример запроса:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <IAM-токен>" \
  -d '@body.json' \
  https://organization-manager.{{ api-host }}/organization-manager/v1/saml/federations
```

Пример ответа:

```bash
{
 "done": true,
 "metadata": {
  "@type": "type.googleapis.com/yandex.cloud.organization-manager.v1.saml.CreateFederationMetadata",
  "federationId": "ajeobmje4dgj********"
 }
```

В свойстве `federationId` указан идентификатор созданной федерации: сохраните его, он понадобится в дальнейшем.
