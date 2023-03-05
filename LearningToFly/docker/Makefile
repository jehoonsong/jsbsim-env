IMGNAME=gcr.io/nettargets-research-cloud/nmap_l2fly
IMAGE=${IMGNAME}:latest
ENROOTIMG=$(HOME)/.enroot/$(IMGNAME).sqsh

CONTAINER?=nmap_l2fly__${USER}_

CRED_FILE=$(HOME)/.config/gcp_sa_credentials.json

GPU_OPT=--gpus all

IPADDR=$(shell hostname)

JUP_PORT=$(shell docker ps | grep ${CONTAINER} | grep -o '[0-9]\+->8888' | grep -o '^[0-9]\+' )

UID=$(shell id -u)
GID=$(shell id -g)
GID_DOCKER=$(shell getent group docker | awk -F: '{printf $$3}')

PROJ_ROOT=../../../..

VOLS= --volume $(HOME):$(HOME) --volume /home/dvc:/home/dvc \
	--volume "/var/run/docker.sock:/var/run/docker.sock:rw" \
	--volume "/etc/group:/etc/group:ro" \
	--volume "/etc/passwd:/etc/passwd:ro" \
	--volume "/etc/shadow:/etc/shadow:ro" \
	--volume $(abspath .)/.bashrc:$(HOME)/.bashrc \
	--user ${UID}:${GID} --group-add ${GID_DOCKER}

ENV_OPTS= -e GOOGLE_APPLICATION_CREDENTIALS=${CRED_FILE} \
		-e HOME=$(HOME) -e PYTHONPATH=$(abspath ${PROJ_ROOT}/ntmap) 

XENV_OPTS= -e DISPLAY=:1 --net=host 

JPY_WORKDIR=$(abspath ../..)

all: jupyter

build: 
	@docker build . -t ${IMAGE}

enroot:
	@read -p "PRESS ENTER TO CONTINUE - !!!THIS WILL DELETE PREVIOUS ENROOT IMAGE!!!"
	@mkdir -p ${HOME}/.enroot \
	    && rm -f ${ENROOTIMG} \
	    && enroot import -o ${ENROOTIMG} dockerd://${IMAGE}

push:
	docker push ${IMAGE}

pull:
	docker pull ${IMAGE}

jupyter: 
	docker run --privileged=true -it --rm -P -d -w=$(HOME) ${GPU_OPT} ${VOLS} ${ENV_OPTS} \
			--name ${CONTAINER} ${IMAGE} \
			bash -c "source /etc/bash.bashrc && SHELL=/bin/bash jupyter lab \
			--notebook-dir=${JPY_WORKDIR} --ip 0.0.0.0 --no-browser --allow-root"
bash: 
	@docker run --privileged=true -it --rm -P -w=$(HOME) ${GPU_OPT} ${VOLS} ${ENV_OPTS} ${XENV_OPTS} \
			--name ${CONTAINER}_2 ${IMAGE} bash -c "cd `pwd` && bash"

train: 
	docker run --privileged=true -i --rm -P -w=$(HOME) ${GPU_OPT} ${VOLS} ${ENV_OPTS} --name ${CONTAINER} ${IMAGE} \
		bash -c "cd `pwd` && cd ../Training/code && \
		python3 run.py --model TailSitter --controller best_model"

play: 
	docker run --privileged=true -it --rm -P -w=$(HOME) ${GPU_OPT} ${VOLS} ${ENV_OPTS} ${XENV_OPTS} \
		--name ${CONTAINER}_2 ${IMAGE} \
		bash -c "cd `pwd` && cd ../Training/code && \
		python3 run.py --model TailSitter --controller model_final --visualize --play"

stop: 
	@docker stop ${CONTAINER}

url:
	$(eval JUP_PORT=$(shell docker port ${CONTAINER} | \
		grep '8888/tcp -> 0.0.0.0' | grep -o ':[0-9]\+' | grep -o '[0-9]\+'))
	$(eval JUP_TOKEN=$(shell docker exec -it ${CONTAINER} jupyter lab list | \
		grep -o 'token=[[:alnum:]]*'))
	$(eval WEB_URL='http://${IPADDR}:${JUP_PORT}/?${JUP_TOKEN}')
	$(eval WEB_URL2='http://${IPADDR2}:${JUP_PORT}/?${JUP_TOKEN}')
	@echo
	@echo 'IP address for Jupyter lab:'
	@echo ${WEB_URL}
	@echo 

clean: 
	find . -name "*~" -exec rm {} \;


