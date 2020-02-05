<template>
	<div>
		<!-- <input
			v-bind:class="{
				'uk-width-1-1': true,
				'uk-range': is_range,
				'uk-input': !is_range
			}"
			v-bind:style="
				is_range ? { paddingRight: padding + 'px', paddingLeft: '5px' } : ''
			"
			v-bind:placeholder="options.placeholder || null"
			v-bind:type="is_range ? 'range' : 'number'"
			v-bind:min="model.min"
			v-bind:max="model.max"
			v-bind:step="model.step"
			v-model.number="model.value"
		/> -->

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
				plugin: 'ef-sbe-image-focus',
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
			// console.log('sbe-number-range', 'plugin:created', this.options);
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
