# How to update private docker registry

## When is the update needed? What need to be updated?

The update of local docker registry is required when the version of Controller used in Windows CI is updated to a newer one. All Contrail images required by Contrail Ansible Deployer running in Windows CI have to be updated to version matching selected Controller version. Some new cached Openstack images may be required by updated Contrail Ansible Deployer.

## How to update

### How to update Contrail images

1. Choose Contrail version tag you want to be cached in private docker registry. All available tags can be found [here](https://hub.docker.com/r/opencontrailnightly/contrail-vrouter-agent/tags/).
2. Log into `winci-registry` VM as root. Check if there is `update-docker-registry.py` script in root directory.
3. If there is no such script, copy it there. The script itself can be found [here](https://github.com/Juniper/contrail-windows-ci/tree/development/utility/update_docker_registry).
4. Run the script with Contrail version tag as parameter, for example:

    ```
    ./update-docker-registry.py ocata-master-206
    ```

5. If new version of Contrail Ansible Deployer uses more images than listed in `update-docker-registry.py` script, please update this list with new images (please update both versions - the one on `winci-registry` VM as well as the one in the repository) and run it again to cache new images.


### How to update Openstack images

New version of Contrail Ansible Deployer may also use some new Openstack images. Currently there is no script that does this automatically. For caching Openstack images manually please do the folowing:

1. Log into `winci-registry` VM as root.
2. Pull required image (where `<IMAGE-NAME>` is the name of the image):

    ```
    docker image pull ci-repo.englab.juniper.net:5000/kolla/<IMAGE-NAME>:ocata
    ```

3. Check the Image ID of previously pulled image:

    ```
    docker image ls | grep kolla | grep <IMAGE-NAME>
    ```

4. Tag the image (replace `<IMAGE-ID>` and `<IMAGE-NAME>` with proper values):

    ```
    docker tag <IMAGE-ID> localhost:5000/kolla/<IMAGE-NAME>:ocata
    ```

5. Push the image to local repository:

    ```
    docker push localhost:5000/kolla/<IMAGE-NAME>:ocata
    ```

6. Repeat steps 2-5 for all other required images.
