1. Make sure $TOFND\_HOME variable is set in .zshrc file
2. Kill Vald and TOFND`
   pkill -9 -f "vald"
   pkill -f "tofnd"`
3. &#x20;Rotate tofnd mnemonic, the new mnemonic is exported automatically

   `tofnd -m rotate -d $TOFND_HOME`
4. Back-UP and encrypt locally the file generated at: $TOFND\_HOME/export
5. Delete file $TOFND\_HOME/export once BACKED-UP, ENCRYPTED, & STORED
6. Run the validator tools script again replacing the below flags as appropriate:

   `./scripts/validator-tools-host.sh -a v0.19.3 -q v0.10.1 -d ~/.axelar_testnet -n testnet -e host -c axelar-testnet-lisbon-3`
