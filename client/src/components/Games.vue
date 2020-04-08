<template>
  <b-container>
		<b-row>
			<b-col col sm="10">
				<h1>Games</h1>
        <hr><br><br>
				<alert :message=message v-if="showMessage"></alert>
        <button type="button" class="btn btn-success btn-sm" v-b-modal.game-modal>Add Game</button>
        <br><br>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Title</th>
              <th scope="col">Publisher</th>
              <th scope="col">Played?</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(game, index) in games" :key="index">
              <td>{{ game.title }}</td>
              <td>{{ game.publisher }}</td>
              <td>
                <span v-if="game.played">Yes</span>
                <span v-else>No</span>
              </td>
              <td>
                <div class="btn-group" role="group">
									<button type="button" class="btn btn-warning btn-sm" v-b-modal.game-update-modal @click="editGame(game)">Update</button>
                  <button type="button" class="btn btn-danger btn-sm" @click="onDeleteGame(game)">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
		</b-col>
	</b-row>

    <b-modal ref="addGameModal"
             id="game-modal"
             title="Add a new game"
             hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-title-group"
                      label="Title:"
                      label-for="form-title-input">
            <b-form-input id="form-title-input"
                          type="text"
                          v-model="addGameForm.title"
                          required
                          placeholder="Enter title">
            </b-form-input>
				</b-form-group>
        <b-form-group id="form-publisher-group"
                      label="Publisher:"
                      label-for="form-publisher-input">
            <b-form-input id="form-publisher-input"
                          type="text"
                          v-model="addGameForm.publisher"
                          required
                          placeholder="Enter publisher">
            </b-form-input>
        </b-form-group>
      <b-form-group id="form-played-group">
        <b-form-checkbox-group v-model="addGameForm.played" id="form-checks">
          <b-form-checkbox value="true">Played?</b-form-checkbox>
        </b-form-checkbox-group>
      </b-form-group>
      <b-button type="submit" variant="primary">Submit</b-button>
      <b-button type="reset" variant="danger">Reset</b-button>
      </b-form>
    </b-modal>

		<!-- edit games -->
		<b-modal ref="editGameModal"
						 id="game-update-modal"
						 title="Update"
						 hide-footer>
			<b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
			<b-form-group id="form-title-edit-group"
										label="Title:"
										label-for="form-title-edit-input">
      	<b-form-input id="form-title-edit-input"
                    type="text"
                    v-model="editForm.title"
                    required
                    placeholder="Enter title">
      	</b-form-input>
    	</b-form-group>
    	<b-form-group id="form-publisher-edit-group"
                  	label="Publisher:"
                  	label-for="form-publisher-edit-input">
      	<b-form-input id="form-publisher-edit-input"
                    	type="text"
                    	v-model="editForm.publisher"
                    	required
                    	placeholder="Enter publisher">
      	</b-form-input>
      </b-form-group>
    	<b-form-group id="form-played-edit-group">
      	<b-form-checkbox-group v-model="editForm.played" id="form-checks">
        	<b-form-checkbox value="true">Played?</b-form-checkbox>
      	</b-form-checkbox-group>
    	</b-form-group>
    	<b-button-group>
      	<b-button type="submit" variant="primary">Update</b-button>
      	<b-button type="reset" variant="danger">Cancel</b-button>
    	</b-button-group>
  		</b-form>
		</b-modal>
  </b-container>
</template>

<script>
import axios from 'axios';
import Alert from './Alert.vue';

export default {
	data() {
		return {
			games: [],
			addGameForm: {
				title: '',
				publisher: '',
				played: [],
			},
			editForm: {
				id: '',
				title: '',
				publisher: '',
				played: [],
			},
			message: '',
			showMessage: false,
		};
	},
	components: {
		alert: Alert,
	},

	methods: {
		getGames() {
			const path = 'http://localhost:5000/games';
			axios.get(path)
				.then((res) => {
					this.games = res.data.games;
				})
				.catch((error) => {
					// eslint-disable-next-line
					console.error(error);
				});
		},
		addGame(payload) {
			const path = 'http://localhost:5000/games';
			axios.post(path, payload)
				.then(() => {
					this.getGames();
					this.message = 'Game added!';
					this.showMessage = true;
				})
				.catch((error) => {
					// eslint-disable-next-line
					console.log(error);
					this.getGames();
					this.message = 'Something went wrong, please try again';
					this.showMessage = true;
				});
		},
		initForm() {
			this.addGameForm.title = '';
			this.addGameForm.publisher = '';
			this.addGameForm.played = [];
			this.editForm.id = '';
			this.editForm.title = '';
			this.editForm.publisher = '';
			this.editForm.read = [];
		},
		onSubmit(evt) {
			evt.preventDefault();
			this.$refs.addGameModal.hide();
			let played = false;
			if (this.addGameForm.played[0]) played = true;
			const payload = {
				title: this.addGameForm.title,
				publisher: this.addGameForm.publisher,
				played, // property shorthand
			};
			this.addGame(payload);
			this.initForm();
		},
		onReset(evt) {
			evt.preventDefault();
			this.$refs.addGameModal.hide();
			this.initForm();
		},
		editGame(game) {
			this.editForm = game;
		},
		removeGame(gameID) {
			const path = `http://localhost:5000/games/${gameID}`;
			axios.delete(path)
				.then(() => {
					this.getGames();
					this.message = 'Game removed!';
					this.showMessage = true;
				})
				.catch((error) => {
					// eslint-disable-next-line
					console.error(error);
					this.getGames();
				});
		},
		onDeleteGame(game) {
			this.removeGame(game.id);
		},
		onSubmitUpdate(evt) {
			evt.preventDefault();
			this.$refs.editGameModal.hide();
			let played = false;
			if (this.editForm.played[0]) played = true;
			const payload = {
				title: this.editForm.title,
				publisher: this.editForm.publisher,
				played,
			};
			this.updateGame(payload, this.editForm.id);
		},
		updateGame(payload, gameID) {
			const path = `http://localhost:5000/games/${gameID}`;
			axios.put(path, payload)
				.then(() => {
					this.getGames();
					this.message = 'Game updated!';
					this.showMessage = true;
				})
				.catch((error) => {
					// eslint-disable-next-line
					console.error(error);
					this.getGames();
				});
		},
		onResetUpdate(evt) {
			evt.preventDefault();
			this.$refs.editGameModal.hide();
			this.initForm();
			this.getGames();
		},
		created() {
			this.getGames();
		},
	},
};
</script>
