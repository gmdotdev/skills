---
name: cobra-cli
description: Scaffold, structure, and document Cobra CLI applications in Go. Use when generating new CLI commands, adding subcommands, wiring flags/args, integrating Viper config, generating shell completions, or reviewing CLI project layout. Triggers on tasks involving Go CLI tools, cobra-cli scaffolding, command hierarchy design, flag/arg patterns, or Cobra best practices.
---

# Cobra CLI Scaffolding

Generate production-quality Cobra CLI command structures, flags, configuration, and documentation for Go projects.

## When to Apply

Use this skill when:
- Creating a new CLI application with Cobra
- Adding commands or subcommands to an existing Cobra project
- Wiring persistent/local flags and argument validation
- Integrating Viper for 12-factor configuration (flags > env > config file > defaults)
- Generating shell completion scripts
- Reviewing or refactoring CLI project structure

## Project Structure

A well-organized Cobra project follows this layout:

```
myapp/
├── main.go              # Minimal: calls cmd.Execute()
├── cmd/
│   ├── root.go          # Root command, global flags, cobra.OnInitialize
│   ├── serve.go         # Example subcommand
│   └── version.go       # Version subcommand
├── internal/            # Business logic (not in cmd/)
├── go.mod
└── go.sum
```

**Key principle:** `cmd/` files define command structure, flags, and argument validation only. Business logic lives in `internal/` or dedicated packages.

## Command Patterns

### Root Command (`cmd/root.go`)

```go
var rootCmd = &cobra.Command{
    Use:   "myapp",
    Short: "One-line description",
    Long:  `Extended description with examples and context.`,
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}

func init() {
    cobra.OnInitialize(initConfig)
    rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "",
        "config file (default $HOME/.myapp.yaml)")
    rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false,
        "verbose output")
}
```

### Subcommand Pattern

```go
var serveCmd = &cobra.Command{
    Use:   "serve",
    Short: "Start the HTTP server",
    Long:  `Start the HTTP server on the specified port.`,
    Example: `  myapp serve --port 8080
  myapp serve --port 3000 --verbose`,
    RunE: func(cmd *cobra.Command, args []string) error {
        port, _ := cmd.Flags().GetInt("port")
        return server.Start(port)
    },
}

func init() {
    rootCmd.AddCommand(serveCmd)
    serveCmd.Flags().IntP("port", "p", 8080, "port to listen on")
}
```

### Argument Validation

| Validator | Use Case |
|-----------|----------|
| `cobra.NoArgs` | Command takes no arguments |
| `cobra.ExactArgs(n)` | Exactly n arguments required |
| `cobra.MinimumNArgs(n)` | At least n arguments |
| `cobra.MaximumNArgs(n)` | At most n arguments |
| `cobra.RangeArgs(min, max)` | Between min and max arguments |
| Custom `Args` func | Domain-specific validation |

## Flag Patterns by Priority

### 1. Persistent Flags (global, inherited by subcommands)

```go
rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file path")
rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false, "verbose output")
rootCmd.PersistentFlags().StringVar(&outputFormat, "output", "text", "output format (text|json|yaml)")
```

### 2. Local Flags (command-specific)

```go
serveCmd.Flags().IntVarP(&port, "port", "p", 8080, "server port")
serveCmd.Flags().BoolVar(&tlsEnabled, "tls", false, "enable TLS")
```

### 3. Required Flags

```go
createCmd.Flags().StringVarP(&name, "name", "n", "", "resource name")
createCmd.MarkFlagRequired("name")
```

### 4. Flag Groups (mutually exclusive or co-dependent)

```go
cmd.MarkFlagsMutuallyExclusive("json", "yaml")
cmd.MarkFlagsRequiredTogether("tls-cert", "tls-key")
```

## Viper Integration (12-Factor Config)

Precedence: Flags > Environment Variables > Config File > Defaults.

```go
func initConfig() {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        home, _ := os.UserHomeDir()
        viper.AddConfigPath(home)
        viper.AddConfigPath(".")
        viper.SetConfigName(".myapp")
        viper.SetConfigType("yaml")
    }
    viper.SetEnvPrefix("MYAPP")
    viper.AutomaticEnv()
    viper.SetEnvKeyReplacer(strings.NewReplacer("-", "_", ".", "_"))
    _ = viper.ReadInConfig()
}
```

Bind flags to viper in `PersistentPreRunE`:

```go
rootCmd.PersistentPreRunE = func(cmd *cobra.Command, args []string) error {
    return viper.BindPFlags(cmd.Flags())
}
```

## Best Practices

| Practice | Rationale |
|----------|-----------|
| Use `RunE` over `Run` | Lets Cobra handle errors consistently |
| One command per file | Maintainability at scale |
| Validate early | Check args/flags before work; fail fast with clear messages |
| Provide `Example` field | Users copy-paste from help text |
| Use `cobra.NoArgs` when appropriate | Prevents accidental argument misuse |
| Keep `cmd/` thin | Business logic in `internal/`; commands are just wiring |
| Add shell completion | `rootCmd.AddCommand(completionCmd)` or use Cobra's built-in |

## Shell Completion

Cobra generates completion scripts automatically:

```go
// Built-in: myapp completion bash|zsh|fish|powershell
// No extra code needed if using cobra-cli scaffolding.
// For manual setup:
rootCmd.AddCommand(&cobra.Command{
    Use:   "completion [bash|zsh|fish|powershell]",
    Short: "Generate shell completion script",
    ValidArgs: []string{"bash", "zsh", "fish", "powershell"},
    Args:  cobra.ExactValidArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        switch args[0] {
        case "bash":
            return rootCmd.GenBashCompletion(os.Stdout)
        case "zsh":
            return rootCmd.GenZshCompletion(os.Stdout)
        case "fish":
            return rootCmd.GenFishCompletion(os.Stdout, true)
        case "powershell":
            return rootCmd.GenPowerShellCompletion(os.Stdout)
        }
        return nil
    },
})
```

## Scaffolding with cobra-cli

```bash
# Install generator
go install github.com/spf13/cobra-cli@latest

# Initialize project
cobra-cli init myapp

# Add commands
cobra-cli add serve
cobra-cli add config
cobra-cli add config show   # nested: config show
```

## Detailed References

For expanded patterns with full code examples:
- `references/testing.md` -- testing Cobra commands
- `references/errors.md` -- error handling patterns
