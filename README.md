# proto_repo

Shared Protocol Buffers (.proto) definitions and generated gRPC Java stubs for the hospital_app system. Services depend on this module to share the contract for inter-service gRPC calls.

Currently contains the UserService gRPC contract used by:
- user_service (gRPC server implementation)
- appointment_service (gRPC client)


## Tech stack
- Protobuf 3.25.2 (protoc)
- gRPC Java 1.65.1 (netty shaded, protobuf, stub)
- Maven protobuf-maven-plugin 0.6.1
- os-maven-plugin (resolves native protoc/protoc-gen-grpc-java artifacts for your OS)
- Java 21


## Module coordinates
- Group: com.hospital_app
- Artifact: proto_repo
- Version: 1.0-SNAPSHOT

Other modules in this monorepo already reference it, for example:

    <dependency>
      <groupId>com.hospital_app</groupId>
      <artifactId>proto_repo</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>


## Protos and generated packages
- Source protos: src/main/proto/
  - user.proto: defines UserService and related messages
- Java generation options (from user.proto):
  - option java_package = "com.hospital_app.proto.generated.user";
  - option java_multiple_files = true;

Generated classes (examples):
- com.hospital_app.proto.generated.user.UserServiceGrpc (service stub/base)
- com.hospital_app.proto.generated.user.GetUserRequest / GetUserResponse
- com.hospital_app.proto.generated.user.UserExistsRequest / UserExistsResponse

Service methods defined in user.proto:
- DoctorExists(UserExistsRequest) -> UserExistsResponse
- PatientExists(UserExistsRequest) -> UserExistsResponse
- GetUser(GetUserRequest) -> GetUserResponse


## Building and using locally
- Build just this module:

      mvn -f proto_repo/pom.xml clean install -DskipTests

- Or as part of the monorepo build from repository root:

      mvn -q -DskipTests package

The install goal publishes the JAR with generated sources to your local Maven repository (~/.m2), so other modules (user_service, appointment_service) can depend on it.

Dockerfiles for services build this module first to ensure generated classes are available in the container build stage.


## How code generation is configured
- Plugin: org.xolstice.maven.plugins:protobuf-maven-plugin:0.6.1
- protoc artifact: com.google.protobuf:protoc:3.25.2
- gRPC codegen plugin: io.grpc:protoc-gen-grpc-java:1.65.1
- pluginParameter: jakarta=true (emits annotations for jakarta instead of javax)

The os-maven-plugin detects your OS/arch and picks the right native executables for protoc and protoc-gen-grpc-java.

Generated sources are placed under:
- target/generated-sources/protobuf/java
- target/generated-sources/protobuf/grpc-java

Maven automatically adds these directories to the compilation source roots during the build.


## Typical usage patterns
- Server (user_service):
  - Implement the generated base class: UserServiceGrpc.UserServiceImplBase
  - Bind methods (GetUser, DoctorExists, PatientExists)
- Client (appointment_service):
  - Use UserServiceGrpc.newBlockingStub(channel) or newFutureStub(channel)
  - Build requests with builders: GetUserRequest.newBuilder().setUserId("...")

See these project paths for concrete usage:
- Server implementation: user_service/src/main/java/.../infra/adapter/in/grpc/controller/user/UserGrpcController.java
- Client adapter: appointment_service/src/main/java/.../infra/adapter/out/user_service/grpc/UserServiceClientGrpcAdapter.java


## Adding or changing protobuf contracts
1) Edit or add .proto files under src/main/proto/.
2) Ensure the option java_package is set to the desired Java package (keep com.hospital_app.proto.generated.<area> for consistency).
3) Run a local build:

       mvn -f proto_repo/pom.xml clean install

4) Update server/client implementations as needed to reflect new or changed RPCs/messages.

Note: Changing message or service shapes is a breaking change for consumers. Prefer additive changes for backward compatibility when possible.


## Troubleshooting
- protoc not found / plugin errors:
  - Ensure internet access for Maven to download protoc and protoc-gen-grpc-java native artifacts.
  - os-maven-plugin resolves ${os.detected.classifier}; verify your platform is supported.
- Version mismatch between runtime and generated code:
  - Keep gRPC and protobuf versions in dependent services aligned with this module (both use 1.65.1 for gRPC and 3.25.x for protobuf).
- Package name mismatches:
  - Generated classes live under com.hospital_app.proto.generated.user (from option java_package). Update imports accordingly.


## Useful commands
- Generate and install locally: mvn -f proto_repo/pom.xml clean install -DskipTests
- Clean only: mvn -f proto_repo/pom.xml clean
- Inspect generated sources: tree proto_repo/target/generated-sources (or open in IDE)


## Project paths
- POM: proto_repo/pom.xml
- Protos: proto_repo/src/main/proto/
- Generated output (build): proto_repo/target/generated-sources/


## License
This project is part of an educational/portfolio repository. See root-level LICENSE if available.