## Usage

- Clone the `Vagrantfile` on this repo.
- Modify the `N_SLAVE` variable to set the number of **slave** nodes available.
- Fire up the cluster using `vagrant up`.
- Head into `localhost:8080` from your browser then look under `Workers` section to verify whether your slave nodes connected to the master node or not (number of available workers should equals with `N_SLAVE` value).
