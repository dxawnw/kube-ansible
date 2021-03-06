RUNTIME := "ubuntu:16.04-systemd"

build:
	sudo podman build -t centos:7-systemd ./centos-7/.
	sudo podman build -t ubuntu:16.04-systemd ./ubuntu-16.04/.
	sudo podman build -t ubuntu:18.04-systemd ./ubuntu-18.04/.
	sudo podman build -t ansible:systemd ./ansible/.

run:
	[ -d ./ssh ] || mkdir -p ssh
	[ -f ./ssh/id_rsa ] || ssh-keygen -t rsa -P "" -f ./ssh/id_rsa
	[ -f ./ssh/authorized_keys ] || cp ./ssh/id_rsa.pub ./ssh/authorized_keys
	sudo podman network create --disable-dns kubernetes --gateway 172.16.16.1 --subnet 172.16.16.0/24  || true
	sudo podman run -d --name ansible  --security-opt seccomp=unconfined  --privileged --network kubernetes --hostname ansible --ip 172.16.16.15 -v ./ssh:/root/.ssh ansible:systemd
	sudo podman run -d --name master01 --security-opt seccomp=unconfined  --privileged --network kubernetes --hostname master01 --ip 172.16.16.11 -v ./ssh:/root/.ssh $(RUNTIME)
	sudo podman run -d --name master02 --security-opt seccomp=unconfined  --privileged --network kubernetes --hostname master02 --ip 172.16.16.12 -v ./ssh:/root/.ssh $(RUNTIME)
	sudo podman run -d --name master03 --security-opt seccomp=unconfined  --privileged --network kubernetes --hostname master03 --ip 172.16.16.13 -v ./ssh:/root/.ssh $(RUNTIME)
	sudo podman run -d --name worker01 --security-opt seccomp=unconfined  --privileged --network kubernetes --hostname worker01 --ip 172.16.16.14 -v ./ssh:/root/.ssh $(RUNTIME)

clean:
	sudo podman rm -f master01 || true
	sudo podman rm -f master02 || true
	sudo podman rm -f master03 || true
	sudo podman rm -f worker01 || true
	sudo podman rm -f ansible || true
	sudo podman network rm kubernetes || true

deploy:
	podman exec -it ansible apt install git make -y
	podman exec -it ansible rm -rf /usr/local/src/kube-ansible
	podman exec -it ansible git clone https://github.com/buxiaomo/kube-ansible.git /usr/local/src/kube-ansible
	podman cp all.yml ansible:/usr/local/src/kube-ansible/group_vars/all.yml
	podman cp hosts ansible:/usr/local/src/kube-ansible/inventory/hosts
	podman exec -it -w /usr/local/src/kube-ansible ansible make