# Using TPM Simulators For Testing

The go-tpm library requires a TPM for testing. Fortunately, there are
software-based TPMs that can be used without access to TPM hardware. This
document describes how to set up a simulator to run `tpm/tpm_test.go`.

## IBM SW TPM 1.2

Use the following commands to set up the IBM SW TPM 1.2 for testing.

```bash
$ TEMP_DIR="$(mktemp -d)"
$ pushd "${TEMP_DIR}"
$ wget https://downloads.sourceforge.net/project/ibmswtpm/tpm4769tar.gz
$ mkdir swtpm
$ tar -C swtpm -xzf tpm4769tar.gz
$ cd swtpm/tpm
$ INSTANCE_CCFLAGS="-UTPM_TEST -DTPM_UNIX_DOMAIN_SOCKET" make -f makefile-en-ac
$ TPM_PORT=socket TPM_PATH=$(pwd) ./tpm_server &>out &
$ export TPM_PATH="$(pwd)/socket"
$ cd "${GO_TPM_PATH}/examples/tpm-startup"
$ go build
$ ./tpm-startup -tpm "${TPM_PATH}"
$ cd ../tpm-takeownership/
$ go build
$ export TPM_OWNER_AUTH=fake
$ export TPM_SRK_OWNER_AUTH=fake
$ ./tpm-takeownership -tpm "${TPM_PATH}"
$ cd ../tpm-genaik/
$ go build
$ export TPM_AIK_AUTH=fake
$ ./tpm-genaik -tpm "${TPM_PATH}" -blob aikblob
$ cp aikblob ../../tpm
$ popd
```

The only commands that fail in the following test (run from the `tpm`
subdirectory) should be the ones that don't have sufficient internal resources
in the test. These can be tested separately by stopping the TPM simulator,
removing 00.permall and socket, and starting it again, then running only the
one test with `go test -v -run <name>`.

```
$ go test -v
```

