---
title: Error Handling Patterns
---

# Error Handling in Cobra Commands

## Use RunE, Not Run

Always prefer `RunE` to propagate errors to Cobra's error handling:

**Incorrect: swallowing errors in Run**

```go
Run: func(cmd *cobra.Command, args []string) {
    if err := doWork(); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)  // bypasses Cobra's cleanup
    }
}
```

**Correct: returning errors from RunE**

```go
RunE: func(cmd *cobra.Command, args []string) error {
    if err := doWork(); err != nil {
        return fmt.Errorf("failed to process: %w", err)
    }
    return nil
}
```

## Silence Usage on Runtime Errors

By default, Cobra prints usage text on any error. For runtime errors (not usage errors), suppress this:

```go
rootCmd.SilenceUsage = true
```

Or per-command:

```go
var serveCmd = &cobra.Command{
    Use:          "serve",
    SilenceUsage: true,
    RunE: func(cmd *cobra.Command, args []string) error {
        // Runtime errors won't print usage
        return server.Start()
    },
}
```

## PreRunE for Validation

Use `PreRunE` to validate preconditions before the main run:

```go
var deployCmd = &cobra.Command{
    Use:   "deploy",
    Short: "Deploy the application",
    PreRunE: func(cmd *cobra.Command, args []string) error {
        token := viper.GetString("api-token")
        if token == "" {
            return fmt.Errorf("--api-token or MYAPP_API_TOKEN is required")
        }
        return nil
    },
    RunE: func(cmd *cobra.Command, args []string) error {
        return deploy.Run(viper.GetString("api-token"))
    },
}
```

## Custom Argument Validators

```go
var processCmd = &cobra.Command{
    Use:  "process [file]",
    Args: func(cmd *cobra.Command, args []string) error {
        if len(args) != 1 {
            return fmt.Errorf("requires exactly one file argument")
        }
        if _, err := os.Stat(args[0]); os.IsNotExist(err) {
            return fmt.Errorf("file %q does not exist", args[0])
        }
        return nil
    },
    RunE: func(cmd *cobra.Command, args []string) error {
        return process.File(args[0])
    },
}
```

## Exit Codes

Map error types to exit codes in `main.go`:

```go
func main() {
    if err := cmd.NewRootCmd().Execute(); err != nil {
        switch {
        case errors.Is(err, cmd.ErrInvalidConfig):
            os.Exit(2)
        case errors.Is(err, cmd.ErrPermission):
            os.Exit(3)
        default:
            os.Exit(1)
        }
    }
}
```
