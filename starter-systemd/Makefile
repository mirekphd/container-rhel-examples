CONTEXT = acme
VERSION = v3.7
IMAGE_NAME = starter-systemd
REGISTRY = docker-registry.default.svc.cluster.local:5000
OC_USER=developer
OC_PASS=developer

# Allow user to pass in OS build options
ifeq ($(TARGET),centos7)
	DFILE := Dockerfile.${TARGET}
else
	TARGET := rhel7
	DFILE := Dockerfile
endif

all: build
build:
	docker build --pull -t ${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION} -t ${CONTEXT}/${IMAGE_NAME} -f ${DFILE} .
	@if docker images ${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION}; then touch build; fi

lint:
	dockerfile_lint -f Dockerfile
	dockerfile_lint -f Dockerfile.centos7
	dockerfile_lint -f Dockerfile.arbuid
	dockerfile_lint -f Dockerfile.arbuid.centos7
	dockerfile_lint -f Dockerfile.w-postgresql

test:
#	OpenShift compatibility - prepare for mount of specified VOLUMEs in image
	$(eval VOL_LIST=$(shell docker inspect -f '{{range $$p, $$vol := .Config.Volumes}} {{json $$p}} {{end}}' ${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION}))
	$(eval TMPFS=$(shell for i in ${VOL_LIST}; do VOLS="$${VOLS} --tmpfs $${i}:rw,suid,dev,exec,relatime,mode=2770"; done; echo $${VOLS}))
	$(eval CONTAINERID=$(shell docker run -tdi \
	-v /sys/fs/cgroup:/sys/fs/cgroup:ro \
	${TMPFS} \
	--cap-drop=MKNOD \
	--cap-drop=SYS_CHROOT \
	${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION}))
	@sleep 15
	@echo ""
	@echo "Check httpd..."
	@docker exec ${CONTAINERID} curl localhost
	@echo ""
	@echo "Check systemd state..."
	@docker exec ${CONTAINERID} systemctl
	@docker exec ${CONTAINERID} journalctl -ax
	@docker exec ${CONTAINERID} systemctl status
	@echo ""
	@echo "Check volume(s)..."
	@for i in ${VOL_LIST}; do docker exec ${CONTAINERID} mountpoint $${i}; docker exec ${CONTAINERID} mount | grep -w $${i} | grep tmpfs; echo; done
	@docker rm -vf ${CONTAINERID}

openshift-test:
	$(eval PROJ_RANDOM=test-$(shell shuf -i 100000-999999 -n 1))
	oc login --token=${OC_PASS}
	oc new-project ${PROJ_RANDOM}
	oc login -u system:admin
	oc patch scc hostmount-anyuid --patch '{"priority":1}'
	oc adm policy add-scc-to-user hostmount-anyuid -z default
	docker login -u ${OC_USER} -p ${OC_PASS} ${REGISTRY}
	docker tag ${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION} ${REGISTRY}/${PROJ_RANDOM}/${IMAGE_NAME}
	docker push ${REGISTRY}/${PROJ_RANDOM}/${IMAGE_NAME}
	oc login --token=${OC_PASS}
	oc new-app -i ${IMAGE_NAME}
	oc rollout status -w dc/${IMAGE_NAME}
	oc volume dc/${IMAGE_NAME} --add -t hostPath --path=/sys/fs/cgroup --mount-path=/sys/fs/cgroup
	oc rollout status -w dc/${IMAGE_NAME}
	oc describe pod ${IMAGE_NAME}-2-
	curl `oc get svc/${IMAGE_NAME} --template '{{.spec.clusterIP}}:{{index .spec.ports 0 "port"}}'`
	oc exec `oc describe pod ${IMAGE_NAME}-2- | grep ^Name\: | grep ${IMAGE_NAME}-2- | sed 's/^.*://g' | tr -d ' \t\n\r\f'` systemctl status

run:
	$(eval VOL_LIST=$(shell docker inspect -f '{{range $$p, $$vol := .Config.Volumes}} {{json $$p}} {{end}}' ${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION}))
	$(eval TMPFS=$(shell for i in ${VOL_LIST}; do VOLS="$${VOLS} --tmpfs $${i}:rw,suid,dev,exec,relatime,mode=2770"; done; echo $${VOLS}))
	docker run -tdi -p 8080:80 \
	-v /sys/fs/cgroup:/sys/fs/cgroup:ro \
	${TMPFS} \
	--cap-drop=MKNOD \
	--cap-drop=SYS_CHROOT \
	${CONTEXT}/${IMAGE_NAME}:${TARGET}-${VERSION}

clean:
	rm -f build