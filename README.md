# lhssh - Local Host SSH Scanner and Connection Manager

[![Version](https://img.shields.io/badge/version-2.0.0-blue)](https://github.com/Open-Technology-Foundation/lhssh)
[![License](https://img.shields.io/badge/license-GPL-3-green)](LICENSE)

A modern, dependency-free bash utility for scanning and managing SSH connections to hosts on local networks.

## Features

- **Fast Network Scanning**: Automatically discover SSH-enabled hosts on your local network
- **Parallel Scanning**: Scan multiple hosts simultaneously for faster results
- **Flexible Connection**: Connect using full IPs or short notation (e.g., `lhssh 152`)
- **Command Execution**: Run commands on remote hosts directly
- **Colored Output**: Easy-to-read results with optional color coding
- **Configurable**: Customize scan ranges, timeouts, and display preferences
- **No External Dependencies**: Pure bash implementation (beyond standard SSH tools)

## Requirements

- Bash 5.0 or later
- SSH client (OpenSSH)
- Either `ssh-keyscan` OR `nc` (netcat) for network scanning
- `timeout` command (part of GNU coreutils)
- Standard Unix tools: `sed`, `grep`, `cut`, `sort`, `xargs`

## Installation

### Manual Installation

1. Clone or download the repository:
```bash
git clone https://github.com/Open-Technology-Foundation/lhssh.git
cd lhssh
```

2. Make the scripts executable:
```bash
chmod +x lhssh lhssh-cmd
```

3. Create symlinks in your PATH:
```bash
sudo ln -s $(pwd)/lhssh /usr/local/bin/lhssh
sudo ln -s $(pwd)/lhssh-cmd /usr/local/bin/lhssh-cmd
```

### Alternative Installation

Copy the scripts to a directory in your PATH:
```bash
sudo cp lhssh lhssh-cmd /usr/local/bin/
sudo chmod +x /usr/local/bin/lhssh /usr/local/bin/lhssh-cmd
```

## Quick Start

1. **Scan your network for SSH hosts:**
```bash
lhssh
```

2. **Connect to a host using short notation:**
```bash
lhssh 152  # Connects to 192.168.1.152
```

3. **Run a command on a remote host:**
```bash
lhssh 152 uptime
```

4. **Show only IP addresses:**
```bash
lhssh -s  # Show full IPs
lhssh -p  # Show last octet only
```

## Usage

```
lhssh [OPTIONS] [IP [COMMAND...]]
```

### Options

#### Display Options
- `-s, --short` - Show IP addresses only (no hostnames)
- `-p, --supershort` - Show only last octet of IP addresses
- `-C, --no-color` - Disable colored output
- `-v, --verbose` - Enable verbose output (use -vv for debug)
- `-q, --quiet` - Disable all non-essential output

#### Network Options
- `-n, --network PREFIX` - Set network prefix (default: 192.168.1.)
- `-b, --begin IP` - Start IP for scanning (default: 50)
- `-f, --finish IP` - End IP for scanning (default: 230)

#### SSH Options
- `-u, --user USERNAME` - SSH username (default: current user)
- `-t, --timeout SECS` - Connection timeout (default: 10)
- `-T, --session-time SECS` - Session timeout (default: 600)

#### Configuration
- `-l, --list` - Show current configuration
- `-e, --edit` - Edit configuration file
- `-S, --save-config` - Save current options to configuration

#### Other
- `-h, --help` - Show help message
- `-V, --version` - Show version information

### Examples

**Scan a different network range:**
```bash
lhssh -n 10.0.0. -b 1 -f 50
```

**Quick scan with short output:**
```bash
lhssh -p
```

**Connect as different user:**
```bash
lhssh -u admin 152
```

**Execute command on all discovered hosts:**
```bash
for ip in $(lhssh -p); do
    echo "=== Host .$ip ==="
    lhssh $ip hostname -f
done
```

**Use with lhssh-cmd for parallel execution:**
```bash
lhssh-cmd "df -h"  # Run on all discovered hosts
```

## Configuration

lhssh stores its configuration in `~/.lhssh.conf`. You can edit this file directly or use the built-in options.

### Configuration Variables

```bash
# Network prefix (must end with dot)
LOCALHOST_HEAD='192.168.1.'

# IP range to scan (last octets)
LOCALHOST_START_IP=50
LOCALHOST_END_IP=230

# SSH login username
LOGIN_USERNAME='root'

# Display preferences
SHORT_DISPLAY=0        # 0=detailed, 1=IPs only
SUPER_SHORT=0         # 0=full IPs, 1=last octet only

# Timeouts (in seconds)
SSH_CONNECT_TIMEOUT=10
SSH_SESSION_TIMEOUT=600

# Features
COLOR_OUTPUT=1        # 0=disable, 1=enable
PARALLEL_SCAN=1       # 0=sequential, 1=parallel
```

### Creating Default Configuration

Generate a new configuration file with default values:
```bash
lhssh -S
```

### Editing Configuration

```bash
lhssh -e  # Opens in default editor
```

## Advanced Usage

### Combining Short Options

lhssh supports combining short options:
```bash
lhssh -vp  # Verbose + supershort display
```

### Custom Network Scanning

Scan a specific range on your current network:
```bash
lhssh -b 100 -f 200  # Scan .100 to .200
```

### Batch Operations

Run commands on multiple specific hosts:
```bash
for host in 152 153 160; do
    lhssh $host "systemctl status sshd"
done
```

### Integration with Other Tools

Use lhssh output with other commands:
```bash
# Find hosts with specific service
lhssh -p | xargs -I{} sh -c 'lhssh {} "systemctl is-active nginx" 2>/dev/null | grep -q active && echo {}'

# Copy file to all hosts
for ip in $(lhssh -p); do
    scp myfile.txt root@192.168.1.$ip:/tmp/
done
```

## Troubleshooting

### No hosts found

1. **Check network connectivity:**
```bash
ip addr show
ping 192.168.1.1
```

2. **Verify SSH service on target hosts:**
```bash
systemctl status sshd  # On target host
```

3. **Test scanning tools:**
```bash
which ssh-keyscan nc
ssh-keyscan -T 1 192.168.1.1
```

4. **Try verbose mode:**
```bash
lhssh -v
```

### Connection issues

1. **Check SSH keys:**
```bash
ssh-keygen -t rsa  # Generate if needed
ssh-copy-id user@host  # Copy to target
```

2. **Verify username:**
```bash
lhssh -u correctuser 152
```

3. **Increase timeout:**
```bash
lhssh -t 30 152  # 30 second timeout
```

### Performance issues

1. **Disable parallel scanning:**
```bash
# Edit config
lhssh -e
# Set PARALLEL_SCAN=0
```

2. **Reduce scan range:**
```bash
lhssh -b 100 -f 150  # Smaller range
```

## Security Considerations

- lhssh uses key-based authentication only (`PasswordAuthentication=no`)
- Configuration file is created with 600 permissions
- SSH host key checking is set to `accept-new` for convenience

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Version History

### v2.0.0 (Current)
- Complete refactor to remove external dependencies
- Added parallel scanning support
- Improved error handling and logging
- Enhanced configuration system
- Better shellcheck compliance

### v1.0.0
- Initial release with nmap dependency
- Basic scanning and connection features

## License

GPL-3 License - see LICENSE file for details

## Support

For issues, questions, or contributions:

- GitHub Issues: [https://github.com/Open-Technology-Foundation/lhssh/issues](https://github.com/Open-Technology-Foundation/lhssh/issues)
- Email: admin@yatti.id
