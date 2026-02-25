---
title: Testing Cobra Commands
---

# Testing Cobra Commands

## Unit Testing Individual Commands

Test commands by executing them programmatically and capturing output:

```go
package cmd_test

import (
    "bytes"
    "testing"

    "github.com/yourorg/myapp/cmd"
)

func executeCommand(args ...string) (string, error) {
    buf := new(bytes.Buffer)
    root := cmd.NewRootCmd()
    root.SetOut(buf)
    root.SetErr(buf)
    root.SetArgs(args)
    err := root.Execute()
    return buf.String(), err
}

func TestServeRequiresValidPort(t *testing.T) {
    _, err := executeCommand("serve", "--port", "-1")
    if err == nil {
        t.Fatal("expected error for invalid port")
    }
}

func TestVersionOutput(t *testing.T) {
    out, err := executeCommand("version")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if out == "" {
        t.Fatal("expected version output")
    }
}
```

**Key pattern:** Export a `NewRootCmd()` constructor instead of a package-level `rootCmd` variable. This allows each test to get a fresh command tree without shared state.

```go
// cmd/root.go
func NewRootCmd() *cobra.Command {
    cmd := &cobra.Command{
        Use:   "myapp",
        Short: "My application",
    }
    cmd.AddCommand(newServeCmd())
    cmd.AddCommand(newVersionCmd())
    return cmd
}

// main.go
func main() {
    if err := cmd.NewRootCmd().Execute(); err != nil {
        os.Exit(1)
    }
}
```

## Testing Flag Parsing

```go
func TestServeFlagDefaults(t *testing.T) {
    root := cmd.NewRootCmd()
    root.SetArgs([]string{"serve"})

    serve, _, _ := root.Find([]string{"serve"})
    port, _ := serve.Flags().GetInt("port")
    if port != 8080 {
        t.Errorf("expected default port 8080, got %d", port)
    }
}
```

## Testing Argument Validation

```go
func TestCreateRejectsNoArgs(t *testing.T) {
    _, err := executeCommand("create")
    if err == nil {
        t.Fatal("expected error when no args provided")
    }
}

func TestCreateAcceptsExactlyOneArg(t *testing.T) {
    _, err := executeCommand("create", "myresource")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}
```

## Integration Testing with Viper

Reset Viper state between tests to avoid cross-contamination:

```go
func TestConfigFromEnv(t *testing.T) {
    t.Setenv("MYAPP_PORT", "9090")
    viper.Reset()

    root := cmd.NewRootCmd()
    root.SetArgs([]string{"serve"})
    _ = root.Execute()

    if viper.GetInt("port") != 9090 {
        t.Errorf("expected port 9090 from env, got %d", viper.GetInt("port"))
    }
}
```
