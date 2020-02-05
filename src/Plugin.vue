<template>
	<div>
		<div class="uk-margin-bottom">
			Source:
			<input
				class="uk-input uk-form-small uk-width-1-1"
				type="text"
				readonly
				v-bind:value="model.src"
			/>
		</div>

		<sb-asset-selector
			v-bind:uid="uid"
			field="src"
			class="uk-button-small uk-margin-bottom"
		/>

		<div class="uk-margin-bottom">
			<img ref="imagePicker" v-bind:src="model.src" />
		</div>
		<div>
			Coordinates:
			<input
				class="uk-input uk-width-1-1"
				type="text"
				readonly
				v-model="coordinates"
			/>
		</div>
	</div>
</template>

<script>
import { FocusPicker } from 'image-focus';

export default {
	mixins: [window.Storyblok.plugin],
	data() {
		return {
			picker: null,
			coordinates: '{x: 0, y: 0}'
		};
	},
	mounted() {
		this.setPicker();
	},
	methods: {
		setPicker() {
			const { imagePicker } = this.$refs;

			this.picker = new FocusPicker(imagePicker, {
				onChange: focus => this.updateModelFocus(focus)
				// focus: {
				// 	x: parseFloat(this.model.x || this.getDefaults().x),
				// 	y: parseFloat(this.model.y || this.getDefaults().y)
				// }
			});
		},
		updateModelFocus(focus) {
			this.model.x = focus.x.toFixed(2);
			this.model.y = focus.y.toFixed(2);
			this.coordinates = `{x: ${this.model.x}, y: ${this.model.y}}`;
		},
		updatePickerFocus(focus) {
			if (this.picker) {
				this.picker.setFocus({
					x: focus.x.toFixed(2),
					y: focus.x.toFixed(2)
				});
			}
		},
		getDefaults() {
			return {
				plugin: 'sbe-image-focus',
				// src: 'https://picsum.photos/2400/1400',
				src: '',
				x: '0.00',
				y: '0.00'
			};
		},
		initWith() {
			// the defaults
			return this.getDefaults();
		},
		pluginCreated() {
			// console.log('sbe-image-focus', 'plugin:created', this.options);
		}
	},
	watch: {
		model: {
			handler(value) {
				this.$emit('changed-model', value);

				console.log('src', this.model.src);
				if (!this.model.src) {
					console.log('null', this.model.src);
				}
			},
			deep: true
		}
	}
};
</script>

<style></style>
