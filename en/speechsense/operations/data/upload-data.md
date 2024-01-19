# Uploading audio data

Use this guide to upload data for API-based speech analysis to {{ speechsense-name }}. This example uses the following parameters:

* Audio format: WAV.
* The dialog metadata is stored in `metadata_example.json`.

An [IAM token](../../../iam/concepts/authorization/iam-token.md) or [IAM key](../../../iam/concepts/authorization/api-key.md) is used to [authenticate](../../api-ref/authentication.md) the service account.

## Getting started {#before-you-begin}

To use the API, you will need Git, Python 3.6 or higher, and the `grpcio-tools` package. [How to install Python](https://www.python.org/downloads/).

1. In the management console, [create a service account](../../../iam/operations/sa/create.md) with the `speech-sense.data.editor` role. To learn more about the roles available in the service, see [{#T}](../../security/index.md).
1. [Add the service account to the namespace](../space/add-user-to-space.md) with the `{{ roles-speechsense-data-editor }}` role. This will authorize the service account to upload data to the [connection](../../concepts/resources-hierarchy.md#connection) you created.
1. [Create an API key](../../../iam/operations/api-key/create.md) or [IAM token](../../../iam/operations/iam-token/create-for-sa.md) for the service account to authenticate with the API.

1. Clone the [{{ yandex-cloud }} API](https://github.com/yandex-cloud/cloudapi) repository:

   ```bash
   git clone https://github.com/yandex-cloud/cloudapi
   ```

1. Install the `grpcio-tools` package using the [pip](https://pip.pypa.io/en/stable/) package manager:

   ```python
   pip install grpcio-tools
   ```

## Uploading data {#upload-data}

1. Go to the folder hosting the {{ yandex-cloud }} API repository, create the `upload_data` folder, generate the client interface code there, and navigate to the `upload_data` folder:

   {% list tabs %}

   - Bash

      ```bash
      cd <path_to_cloudapi_directory> && \
      mkdir upload_data && \
      python3 -m grpc_tools.protoc -I . \
           --python_out=./upload_data/ \
           --grpc_python_out=./upload_data/ \
           yandex/cloud/speechsense/v1/talk_service.proto \
           yandex/cloud/speechsense/v1/audio.proto && \
      cd upload_data
      ```

   {% endlist %}

1. In the `upload_data` folder, create the `upload_grpc.py` Python script to upload your data to a {{ speechsense-name }} connection. You can use a single message to upload your data or transmit the data in parts via a gRPC stream.

   {% list tabs %}

   - Single message

   	```python
   	import argparse
   	import json
   	from typing import Dict
   	import grpc
   	import datetime

   	from yandex.cloud.speechsense.v1 import talk_service_pb2
   	from yandex.cloud.speechsense.v1 import talk_service_pb2_grpc
   	from yandex.cloud.speechsense.v1 import audio_pb2

      # For authorization with IAM token replace api_key for iam_token
      def upload_talk(connection_id: int, metadata: Dict[str, str], api_key: str, audio_bytes: bytes):
         credentials = grpc.ssl_channel_credentials()
         channel = grpc.secure_channel('{{ speechsense-endpoint }}', credentials)
         talk_service_stub = talk_service_pb2_grpc.TalkServiceStub(channel)

      # Generating an API request
         request = talk_service_pb2.UploadTalkRequest(
            metadata=talk_service_pb2.TalkMetadata(
               connection_id=str(connection_id),
               fields=metadata
            ),
            # Audio format: WAV
            audio=audio_pb2.AudioRequest(
               audio_metadata=audio_pb2.AudioMetadata(
                  container_audio=audio_pb2.ContainerAudio(
                     container_audio_type=audio_pb2.ContainerAudio.ContainerAudioType.CONTAINER_AUDIO_TYPE_WAV
                  )
               ),
               audio_data=audio_pb2.AudioChunk(data=audio_bytes)
            )
         )
         # Authorization type: API key
         response = talk_service_stub.Upload(request, metadata=(
            ('authorization', f'Api-Key {api_key}'),
         # For authorization with IAM token supply header
         # ('authorization', f'Bearer {iam_token}'), 
         ))

         # Display dialog ID
         print(f'Dialog ID: {response.talk_id}')

      if __name__ == '__main__':
         parser = argparse.ArgumentParser()
         parser.add_argument('--key', required=True, help='API key or IAM token', type=str)
         parser.add_argument('--connection-id', required=True, help='Connection ID', type=str)
         parser.add_argument('--audio-path', required=True, help='Audio file path', type=str)
         parser.add_argument('--meta-path', required=False, help='JSON with the dialog metadata', type=str, default=None)
         args = parser.parse_args()

         # Default values if no metadata specified
         if args.meta_path is None:
            now = datetime.datetime.now().isoformat()
            metadata = {
               'operator_name': 'Operator',
               'operator_id': '1111',
               'client_name': 'Client',
               'client_id': '2222',
               'date': str(now),
               'date_from': '2023-09-13T17:30:00.000',
               'date_to': '2023-09-13T17:31:00.000',
               'direction_outgoing': 'true',
            }
         else:
            with open(args.meta_path, 'r') as fp:
               metadata = json.load(fp)

         with open(args.audio_path, 'rb') as fp:
            audio_bytes = fp.read()
         upload_talk(args.connection_id, metadata, args.key, audio_bytes)
      ```

   - Streaming

   	```python
   	import argparse
   	import json
   	from typing import Dict
   	import grpc
   	import datetime

   	from yandex.cloud.speechsense.v1 import talk_service_pb2
   	from yandex.cloud.speechsense.v1 import talk_service_pb2_grpc
   	from yandex.cloud.speechsense.v1 import audio_pb2

      # Chunk size is 1 MB
      CHUNK_SIZE_BYTES = 1024

      def upload_audio_requests_iterator(connection_id: int, metadata: Dict[str, str], audio_path: str):
         yield talk_service_pb2.StreamTalkRequest(
            metadata=talk_service_pb2.TalkMetadata(
               connection_id=str(connection_id),
               fields=metadata
            )
         )
         # Read data from the file and send it to the stream
         with open(audio_path, mode='rb') as fp:
            data = fp.read(CHUNK_SIZE_BYTES)
            while data != b'':
               yield talk_service_pb2.StreamTalkRequest(
                  audio=talk_service_pb2.AudioStreamingRequest(
                     audio_metadata=audio_pb2.AudioMetadata(
                        container_audio=audio_pb2.ContainerAudio(
                           container_audio_type=audio_pb2.ContainerAudio.ContainerAudioType.CONTAINER_AUDIO_TYPE_WAV
                        )
                     ),
                     audio_data=audio_pb2.AudioChunk(data=data)
                  )
               )
               data = fp.read(CHUNK_SIZE_BYTES)

      # For authorization with an IAM token, replace api_key with iam_token
      def upload_talk(connection_id: int, metadata: Dict[str, str], api_key: str, audio_path: str):
         credentials = grpc.ssl_channel_credentials()
         channel = grpc.secure_channel('api.talk-analytics.yandexcloud.net:443', credentials)
         talk_service_stub = talk_service_pb2_grpc.TalkServiceStub(channel)

         # Authorization type: API key
         response = talk_service_stub.UploadAsStream(
            upload_audio_requests_iterator(connection_id, metadata, audio_path),
            metadata=(('authorization', f'Api-Key {api_key}'),
         # For authorization with an IAM token, provide the metadata
         # metadata=(('authorization', f'Bearer {iam_token}'),
         ))

         # Output the dialog ID
         print(f'Dialog ID: {response.talk_id}')

      if __name__ == '__main__':
         parser = argparse.ArgumentParser()
         parser.add_argument('--key', required=True, help='API key or IAM token', type=str)
         parser.add_argument('--connection-id', required=True, help='Connection ID', type=str)
         parser.add_argument('--audio-path', required=True, help='Audio file path', type=str)
         parser.add_argument('--meta-path', required=False, help='JSON with the dialog metadata', type=str, default=None)
         args = parser.parse_args()

         # Default values if no metadata is provided
         if args.meta_path is None:
            now = datetime.datetime.now().isoformat()
            metadata = {
               'operator_name': 'Operator',
               'operator_id': '1111',
               'client_name': 'Client',
               'client_id': '2222',
               'date': str(now),
               'date_from': '2023-09-13T17:30:00.000',
               'date_to': '2023-09-13T17:31:00.000',
               'direction_outgoing': 'true',
            }
         else:
            with open(args.meta_path, 'r') as fp:
               metadata = json.load(fp)

         upload_talk(args.connection_id, metadata, args.key, args.audio_path)
      ```

   {% endlist %}

1. Specify the service account's API key:

   ```bash
   export API_KEY=<service_account_API_key>
   ```

   If using an IAM token, provide it instead of the API key:

   ```bash
   export IAM_TOKEN=<service_account_IAM_token>
   ```

1. Run the `upload_grpc.py` script with the following parameters:

   ```bash
   python3 upload_grpc.py \
      --audio-path <audio_file> \
      --meta-path <metadata> \
      --connection-id <connection_ID> \
      --key ${API_KEY}
   ```

   Where:

   * `audio-path`: Path to the audio file with the dialog.
   * `meta-path`: Path to the file with the dialog metadata.
   * `connection-id`: ID of the connection you upload the data to.
   * `key`: API key for authorization. If using an IAM token, specify the `IAM_TOKEN` environment variable instead of `API_KEY`.
