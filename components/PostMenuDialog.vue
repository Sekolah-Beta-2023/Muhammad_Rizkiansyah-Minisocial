<template>
	<v-dialog v-model="dialog" max-width="400px">
		<template #activator="{ on, attrs }">
			<v-btn v-if="$auth.user.id === userId" icon v-bind="attrs" v-on="on">
				<v-icon>mdi-dots-vertical</v-icon>
			</v-btn>
		</template>
		<v-card v-if="$auth.user.id === userId">
			<v-card-title>
				Postingan Anda
			</v-card-title>
			<v-divider/>
			<v-card-text class="pa-0">
				<v-list>
					<v-list-item v-for="(item, i) in listItems" :key="i" @click="item[1]">
						<v-list-item-title>
							{{ item[0] }}
						</v-list-item-title>
					</v-list-item>
				</v-list>
			</v-card-text>
		</v-card>
	</v-dialog>
</template>

<script>
	export default {
		name: 'PostMenuDialog',
		props: {
			userId: String(''),
			postId: String('')
		},
		data() {
			return {
				listItems: [
					["Hapus Postingan", () => this.deletePost()],
					["Ubah Teks Caption", () => this.editPost()],
				],
				dialog: false
			}
		},
		methods: {
			async deletePost() {
				try {
					const res = await this.$swal.fire({
						text: 'Apakah kau ingin menghapus postingan ini? Postingan dihapus tidak bisa kembali',
						icon: 'question',
						confirmButtonText: 'Ya',
						denyButtonText: 'Tidak',
						showDenyButton: true,
					});
					let data;
					if (res.isConfirmed) {
						data = (await this.$axios.$delete(`/posts/${this.postId}`)).data;
					}
					if (data) {
						this.$router.push(`/u/${this.$auth.user.username}`);
					}
				} catch (e) {
					console.error(e);
				}
			},
			async editPost() {
				try {
					this.dialog = false;
					const { value: caption, isConfirmed } = await this.$swal.fire({
						text: 'Edit Caption Postingan Anda : ',
						input: 'textarea',
						inputPlaceholder: 'Tulis caption di sini',
						title: 'Caption Postingan',
					});
					if (!isConfirmed) return null;
					await this.$axios.$put(`/posts/${this.postId}`, { caption });
					this.$router.go(0);
				} catch (e) {
					console.error(e);
				}
			}
		}
	}
</script>