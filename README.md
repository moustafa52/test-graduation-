```bash
# Navigate to the Kubernetes manifests and scripts directory
cd ~/5g-project/k8s

# Start the 5G Core, UERANSIM, and monitoring components
./start-k8s.sh


# List all pods running in the 5G Core namespace and show their current status
kubectl get pods -n 5g-core



# List running NR-UE processes and their command-line arguments inside the UE container
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
pgrep -af '/ueransim/nr-ue'



# Display the last 15 log lines for each of the five configured UEs
for i in 1 2 3 4 5; do
  IMSI="99970000000000$i"
  echo "========== UE-$i ($IMSI) =========="
  kubectl exec -n 5g-core deployment/ueransim -c ue -- \
    sh -c "tail -n 15 /tmp/ue-$IMSI.log 2>/dev/null"
done




# Ping the 5G Core UE gateway (10.45.0.1) through each of the first five UE tunnels
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'for i in 0 1 2 3 4; do
  echo "===== uesimtun$i ====="
  ping -I uesimtun$i -c 4 -W 2 10.45.0.1
done'





# Test internet connectivity to 8.8.8.8 through each of the first five UE tunnels
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'for i in 0 1 2 3 4; do
  echo "===== uesimtun$i ====="
  ping -I uesimtun$i -c 4 -W 3 8.8.8.8
done'





# Display IPv4 addresses assigned to UE tunnel interfaces
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
ip -o -4 addr show | grep uesimtun





# Ping the 5G Core UE gateway through each of the first five UE tunnels, allowing up to 3 seconds per reply
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'for i in 0 1 2 3 4; do
  echo "===== UE Tunnel $i ====="
  ping -I uesimtun$i -c 4 -W 3 10.45.0.1
done'





# Test external internet connectivity through each of the first five UE tunnels, allowing up to 3 seconds per reply
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'for i in 0 1 2 3 4; do
  echo "===== UE Tunnel $i ====="
  ping -I uesimtun$i -c 4 -W 3 8.8.8.8
done'





# Display the last 15 lines of the UE synchronization script log
tail -n 15 /tmp/ue-sync.log





# Check whether the NR-UE process for IMSI 999700000000006 is running inside the UE container
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
pgrep -af 'nr-ue.*999700000000006'





# Show the last 10 log entries matching successful registration, successful PDU session establishment, or related failures for IMSI 999700000000006
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c "grep -E 'Registration is successful|PDU Session establishment is successful|Registration.*failed|PDU Session.*failed' /tmp/ue-999700000000006.log | tail -n 10"





# Display the last 15 lines of the UE synchronization script log again
tail -n 15 /tmp/ue-sync.log





# List NR-UE processes with their process IDs and full command-line arguments inside the UE container
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'ps -eo pid,args | grep "[/]ueransim/nr-ue"'





# Display IPv4 addresses assigned to UE tunnel interfaces
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
ip -o -4 addr show | grep uesimtun





# Test connectivity from UE tunnel 6 to the 5G Core UE gateway and then to the public IP address 8.8.8.8
kubectl exec -n 5g-core deployment/ueransim -c ue -- \
bash -c 'ping -I uesimtun6 -c 4 -W 3 10.45.0.1 && ping -I uesimtun6 -c 4 -W 3 8.8.8.8'
```
