# How to update private docker registry

## When is the update needed? What need to be updated?

The update of local docker registry is required when the version of Controller used in Windows CI is updated to a newer one. All Contrail images required by Contrail Ansible Deployer running in Windows CI have to be updated to version matching selected Controller version. Some new cached Openstack images may be required by updated Contrail Ansible Deployer.

## How to update

### How to update Contrail images

1. Choose Contrail version tag you want to be cached in private docker registry:
    - all available tags can be found [here](https://hub.docker.com/r/opencontrailnightly/contrail-vrouter-agent/tags/).

2. Log into `winci-registry`:
    - please contact Windows CI team for access.

3. Check if there is `update-docker-registry.py` script in root directory.

4. If there is no such script:
    - download it from [here](https://github.com/Juniper/contrail-windows-ci/tree/development/utility/update_docker_registry):

            curl https://raw.githubusercontent.com/Juniper/contrail-windows-ci/development/utility/update_docker_registry/update-docker-registry.py --output update-docker-registry.py

    - make it executable:

            chmod +x ./update-docker-registry.py

5. Run the script with Contrail version tag as parameter, for example:

        ./update-docker-registry.py ocata-master-206

6. If new version of Contrail Ansible Deployer uses more images than listed in `update-docker-registry.py` script:
    - update images list in the script locally,
    - open up a PR with updated images list to contrail-windows-ci.


### How to update Openstack images

New version of Contrail Ansible Deployer may also use some new Openstack images. Currently there is no script that does this automatically. For caching Openstack images manually please do the folowing:

1. Log into `winci-registry`:
    - please contact Windows CI team for access.

2. Set the name of required image in local variable (replace `<IMAGE-NAME>` with proper value:

        image_name=<IMAGE-NAME>

3. Pull the image:

        docker image pull ci-repo.englab.juniper.net:5000/kolla/${image_name}:ocata

4. Get the Image ID of previously pulled image:

        image_id=$(docker image ls | grep kolla | grep ${image_name} | awk '{ print $3 }' | uniq)

    - verify the ID:

            echo "$image_id"

5. Tag the image:

        docker tag ${image_id} localhost:5000/kolla/${image_name}:ocata

6. Push the image to local repository:

        docker push localhost:5000/kolla/${image_name}:ocata

7. Repeat steps 2-6 for all other required images.
