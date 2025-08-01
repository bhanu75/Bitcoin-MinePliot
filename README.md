# Bitcoin Mine Control System 🏭

A comprehensive monitoring and control system for a 1MW remote Bitcoin mining operation, designed to run on Raspberry Pi 4 with automatic power curtailment, temperature management, and extensive logging.

## 🎯 Features

### Core Functionality
- **Automated Power Management**: Responds to utility power signals for demand response
- **Temperature Monitoring**: RoomAlert integration with automated fan control
- **Miner Control**: Foreman.mn API integration for miner management
- **Smart PDU Control**: Automated power distribution unit management
- **Fan System Control**: Individual fan control (1-9 fans) based on temperature

### Monitoring & Alerts
- **Multi-channel Notifications**: Telegram, Email, and SMS alerts
- **Heartbeat Monitoring**: Regular system health checks
- **Extensive Logging**: All actions and state changes logged
- **Time Series Database**: SQLite-based data storage for historical analysis
- **Web Dashboard**: Real-time system status monitoring
- **Debug Dashboard**: Advanced testing and state manipulation tools

### Safety & Reliability
- **Emergency Shutdown**: Automatic system protection
- **Redundant Communications**: Multiple notification channels
- **State Machine Architecture**: Predictable system behavior
- **Automatic Backups**: Regular data and configuration backups
- **Failsafe Defaults**: Safe operation when sensors fail

## 🏗️ System Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Power Signal  │    │  Mine Controller │    │  Notifications  │
│  (cbpower.coop) │───▶│   (Raspberry Pi) │───▶│ Telegram/Email  │
└─────────────────┘    │                  │    │     /SMS        │
                       │  ┌─────────────┐ │    └─────────────────┘
┌─────────────────┐    │  │ State       │ │    
│   RoomAlert     │───▶│  │ Machine     │ │    ┌─────────────────┐
│ (Temp/Humidity) │    │  └─────────────┘ │───▶│   Web Dashboard │
└─────────────────┘    │                  │    └─────────────────┘
                       │  ┌─────────────┐ │    
┌─────────────────┐    │  │ SQLite      │ │    ┌─────────────────┐
│   IotaWatt      │───▶│  │ Database    │ │───▶│ Debug Dashboard │
│ (Fan Wattage)   │    │  └─────────────┘ │    └─────────────────┘
└─────────────────┘    └──────────────────┘    
                              │     │
                              ▼     ▼
                       ┌─────────────────┐ ┌─────────────────┐
                       │  Foreman.mn     │ │  Fan Relays     │
                       │  (Miners)       │ │  (1-9 Fans)     │
                       └─────────────────┘ └─────────────────┘
```

## 📦 Installation

### Quick Install
```bash
# Download and run the installation script
curl -sSL https://raw.githubusercontent.com/yourusername/bitcoin-mine-controller/main/install.sh | bash

# Or manual installation:
git clone https://github.com/yourusername/bitcoin-mine-controller.git
cd bitcoin-mine-controller
chmod +x install.sh
sudo ./install.sh
```

### Manual Setup
1. **System Requirements**
   - Raspberry Pi 4 (8GB recommended)
   - Raspbian OS (latest)
   - Python 3.8+
   - Fixed IP address configuration

2. **Network Configuration**
   ```bash
   # Set static IP for Raspberry Pi
   sudo nano /etc/dhcpcd.conf
   
   # Add your network configuration:
   interface eth0
   static ip_address=192.168.1.50/24
   static routers=192.168.1.1
   static domain_name_servers=192.168.1.1
   ```

3. **Install Dependencies**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3-pip python3-venv git sqlite3 nginx cron
   ```

## ⚙️ Configuration

### Main Configuration File (`config/config.txt`)

```ini
[GENERAL]
loop_interval_minutes = 2
max_temp_celsius = 35
min_temp_celsius = 10
max_humidity = 80

[APIS]
cornbelt_url = https://cbpower.coop/power-signal
roomalert_ip = 192.168.1.100
iotawatt_ip = 192.168.1.101
foreman_api = http://foreman.mn/api
pdu_ip = 192.168.1.102
fan_relay_ip = 192.168.1.103

[NOTIFICATIONS]
telegram_token = YOUR_TELEGRAM_BOT_TOKEN
telegram_chat_id = YOUR_TELEGRAM_CHAT_ID
email_smtp_server = smtp.gmail.com
email_smtp_port = 587
email_username = your_email@gmail.com
email_password = your_app_password
email_recipients = admin@yourdomain.com,backup@yourdomain.com
sms_api_key = YOUR_SMS_API_KEY
sms_phone = +1234567890
```

### Device IP Addresses
Ensure all devices have static IP addresses on your network:

| Device | Default IP | Purpose |
|--------|------------|---------|
| Raspberry Pi | 192.168.1.50 | Main controller |
| RoomAlert | 192.168.1.100 | Temperature/Humidity |
| IotaWatt | 192.168.1.101 | Power monitoring |
| Smart PDU | 192.168.1.102 | Power distribution |
| Fan Relays | 192.168.1.103 | Fan control |

## 🚀 Usage

### Starting the System
```bash
# Start as service (recommended)
sudo systemctl start bitcoin-mine
sudo systemctl enable bitcoin-mine

# Or run manually for testing
cd /opt/bitcoin-mine
source venv/bin/activate
python mine_controller.py

# Run single iteration (cron mode)
python mine_controller.py --cron
```

### Accessing Dashboards
- **Main Dashboard**: http://your-pi-ip/
- **Debug Dashboard**: http://your-pi-ip/debug.html
- **API Status**: http://your-pi-ip/api/status

### Command Line Tools
```bash
# View logs
journalctl -u bitcoin-mine -f
tail -f /opt/bitcoin-mine/logs/mine_controller.log

# Check system status
sudo systemctl status bitcoin-mine

# Test configuration
cd /opt/bitcoin-mine && python test_system.py

# Manual backup
./scripts/backup.sh

# View database
sqlite3 mine_data.db "SELECT * FROM mine_states ORDER BY timestamp DESC LIMIT 10;"
```

## 🔧 API Endpoints

### Status Endpoints
- `GET /api/status` - Current system status
- `GET /api/state` - Detailed system state
- `GET /dashboard_data.json` - Dashboard data feed

### Debug Endpoints (Debug Dashboard Only)
- `POST /api/debug/power-signal` - Override power signal
- `POST /api/debug/miners` - Control miners manually
- `POST /api/debug/fans` - Control fan system
- `POST /api/debug/temperature` - Set temperature override
- `POST /api/debug/raw-state` - Direct state manipulation

## 📊 Monitoring & Logging

### Log Levels
- **INFO**: Normal operations and state changes
- **WARN**: Warning conditions that need attention
- **ERROR**: Error conditions that affect functionality
- **DEBUG**: Detailed debugging information (debug mode only)

### Alert Types
1. **Heartbeat** (Hourly): System is alive and operational
2. **Warning** (Telegram + Email): Non-critical issues requiring attention
3. **Emergency** (All channels): Critical issues requiring immediate action

### Common Alert Scenarios
- High temperature (>35°C)
- Very high temperature (>40°C) - Emergency
- Power signal changes
- API communication failures
- Fan system malfunctions
- Database errors

## 🔒 Security Considerations

### Network Security
- Use VPN access for remote management
- Configure firewall rules appropriately
- Change default passwords on all devices
- Use HTTPS where possible

### API Security
- Debug endpoints should only be accessible locally
- Consider IP whitelisting for critical functions
- Regularly update all system components
- Monitor access logs

## 🛠️ Troubleshooting

### Common Issues

**System not responding to power signals**
```bash
# Check Cornbelt URL accessibility
curl -I https://cbpower.coop/power-signal

# Verify network configuration
ping 8.8.8.8

# Check logs for parsing errors
grep "power signal" /opt/bitcoin-mine/logs/mine_controller.log
```

**Temperature readings incorrect**
```bash
# Test RoomAlert API
curl http://192.168.1.100/api/sensors

# Check device IP configuration
nmap -sn 192.168.1.0/24

# Verify sensor calibration
```

**Notifications not working**
```bash
# Test Telegram bot
curl -X POST "https://api.telegram.org/bot$TOKEN/sendMessage" \
  -d "chat_id=$CHAT_ID&text=Test message"

# Check email configuration
python -c "import smtplib; print('SMTP libraries available')"

# Verify notification logs
grep "notification" /opt/bitcoin-mine/logs/mine_controller.log
```

**Database issues**
```bash
# Check database integrity
sqlite3 mine_data.db "PRAGMA integrity_check;"

# Repair database if needed
sqlite3 mine_data.db ".backup backup.db"

# Reset database (WARNING: Data loss)
rm mine_data.db && python mine_controller.py --init-db
```

### System Recovery
```bash
# Complete system restart
sudo systemctl stop bitcoin-mine
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl start bitcoin-mine

# Emergency shutdown
python -c "
import requests
requests.post('http://localhost:5000/api/debug/emergency-shutdown')
"
```

## 📈 Performance Tuning

### Optimization Tips
1. **Loop Interval**: Adjust based on your power signal update frequency
2. **Database Maintenance**: Regular cleanup of old data
3. **Log Rotation**: Configure appropriate log retention
4. **Network Timeouts**: Tune API timeout values for your network
5. **Memory Management**: Monitor Pi memory usage under load

### Monitoring Performance
```bash
# System resources
htop
df -h
free -h

# Network connectivity
ping -c 4 192.168.1.100  # RoomAlert
ping -c 4 192.168.1.101  # IotaWatt

# Database performance
sqlite3 mine_data.db "ANALYZE; .timer ON; SELECT COUNT(*) FROM mine_states;"
```

## 🔄 Maintenance

### Daily Tasks (Automated)
- Log rotation and cleanup
- Database maintenance
- System health checks
- Backup verification

### Weekly Tasks
- Review alert patterns
- Check system performance
- Update configuration if needed
- Test notification channels

### Monthly Tasks
- Full system backup
- Software updates
- Hardware inspection
- Performance analysis

## 📞 Support

### Getting Help
1. Check the troubleshooting section above
2. Review system logs for error messages
3. Test individual components using debug dashboard
4. Consult API documentation for third-party services

### Contributing
- Report bugs via GitHub issues
- Submit feature requests
- Share configuration improvements
- Contribute to documentation

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Cornbelt Power Cooperative for demand response program
- Foreman.mn for mining management APIs
- RoomAlert for environmental monitoring
- IotaWatt for power monitoring solutions

---

**⚠️ Important Safety Notes:**
- Always test configuration changes in a safe environment
- Monitor system closely during initial deployment
- Ensure proper electrical safety for all installations
- Have emergency shutdown procedures ready
- Keep backup communication methods available

**🔧 System Status:**
- Version: 1.0.0
- Last Updated: January 2025
- Compatibility: Raspberry Pi 4, Python 3.8+
- Dependencies: See requirements.txt
