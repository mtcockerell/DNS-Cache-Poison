DNS Cache Poisoning Attack Demo
===============================

This project is to investigate and reproduce the DNS Cache Poisoning Attack within an isolated network environment using docker. The demo shows an adversary to spoof the DNS answer of google.com and direct to the adversary's malicious IP address.

## How to:

### Prerequisites

You will need Docker and Docker Compose installed for this project.

### Build


```bash
git clone https://github.com/mtcockerell/DNS-Cache-Poison.git
cd DNS-Cache-Poison
docker-compose build
docker-compose up
```
Upon running, the DNS resolver and upstream DNS server will start listening on port 53.

### Attack

#### 1. Open another terminal and start the container `attacker`

```bash
docker exec -it attacker bash
```

#### 2. Launch the attack script

```bash
python attack.py google.com 10.20.30.4
```

#### 3. Impact

![](/screenshots/1.png)

As the output of the original container `dns` shows, a fake DNS record has been successfully written into cache. It can be verified by using dig in the victim's container.

Now start the `victim`'s container in a new terminal:

```bash
docker exec -it victim bash
```

In the new victim container:

```bash
dig google.com
```

![](/screenshots/2.png)

The screenshot should show that in the answer section, the record is pointing to the adversary's IP address (10.20.30.4).

## Results

### Project Structure

Four containers are used in this project:\
`DNS` (`10.20.30.2`): A vulnerable DNS resolver\
`Victim` (`10.20.30.3`): A victim host for proving the attack\
`Attacker` (`10.20.30.4`): The attacker who will launch the attack\
`Upstream DNS` (`10.20.30.5`): Plays the role of an upstream DNS server

### DNS Cache Server (`10.20.30.2`)

This is a simple DNS resolver developed to fully examine the attack process. It first looks at the cache stored and returns the cache if there is. Otherwise, it will forward the request to the upstream DNS server, waiting for responses.

As the objective of this project is only to investigate the attack itself, the Query ID (QID) for requesting is always set to range from 10000 to 10050, and the source port is fixed to 22222 to avoid launching a birthday attack, which is not the goal of this project.

Once it has received the response, it will validate if two QIDs match and then send the answer to the client.

### Upstream DNS (`10.20.30.5`)

A local authoritative DNS is set up to take the place of upstream DNS servers to always respond DNS requests with IP `1.2.3.4`

### Attacker (`10.20.30.4`)

The attacker first sends a DNS query to the DNS cache server (`10.20.30.2`), so the server will send a DNS request to the upstream server and begin accepting responses. Then it sends DNS answers trying all possible QID from the destination impersonating the upstream server (`10.20.30.5`) to the cache server (`10.20.30.2`).

An attack script (`/attacker/attack.py`) is thereby created.

#### Usage:
```bash
python attack.py [target domain] [spoofed IP]
```

## License

 [MIT License](LICENSE). 
