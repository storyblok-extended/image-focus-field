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
		></sb-asset-selector>

		<div class="bordered uk-margin-bottom">
			<div class="wrapper">
				<img ref="imagePicker" class="uk-width-1-1" v-bind:src="model.src" />
			</div>
		</div>

		<div>
			Coordinates:
			<input
				class="uk-input uk-width-1-1"
				type="text"
				readonly
				v-bind:value="coordinates"
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
			coordinates: ''
		};
	},
	mounted() {
		this.setPicker();
		window.addEventListener('resize', this.handleResize);
	},
	beforeDestroy() {
		window.removeEventListener('resize', this.handleResize);
	},
	methods: {
		handleResize() {
			this.updatePickerFocus({
				x: parseFloat(this.model.x || this.getDefaults().x),
				y: parseFloat(this.model.y || this.getDefaults().y)
			});
		},
		setPicker() {
			const { imagePicker } = this.$refs;

			this.picker = new FocusPicker(imagePicker, {
				onChange: focus => this.updateModelFocus(focus),
				focus: {
					x: parseFloat(this.model.x || this.getDefaults().x),
					y: parseFloat(this.model.y || this.getDefaults().y)
				}
			});
		},
		updateModelFocus(focus) {
			// console.log('update model focus', focus.x, focus.y);
			this.model.x = parseFloat(focus.x).toFixed(2);
			this.model.y = parseFloat(focus.y).toFixed(2);
			this.coordinates = `{x: ${this.model.x}, y: ${this.model.y}}`;
		},
		updatePickerFocus(focus) {
			if (this.picker) {
				// console.log('update picker focus', focus.x, focus.y);
				this.picker.setFocus({
					x: parseFloat(focus.x).toFixed(2),
					y: parseFloat(focus.y).toFixed(2)
				});
			}
		},
		getDefaults() {
			return {
				plugin: 'ef-sbe-image-focus',
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
		'model.src': {
			handler() {
				if (!this.model.src) {
					this.model.x = this.getDefaults().x;
					this.model.y = this.getDefaults().y;
				}
			}
		},
		'model': {
			handler(value) {
				this.$emit('changed-model', value);
			},
			deep: true
		}
	}
};
</script>

<style>
.bordered {
	border: 1px solid #e0e0e0;
}
</style>
