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

		<sb-asset-selector v-bind:uid="uid" field="src" class="uk-button-small uk-margin-bottom"></sb-asset-selector>

		<div class="uk-flex uk-flex-space-around uk-margin-bottom">
			<div>
				Dimension:
				<b>{{ originalDimensions }}</b>
			</div>
			<div>
				Natural ratio:
				<b>{{ aspectRatio }}</b>
			</div>
			<div>
				Mode:
				<b>{{ mode }}</b>
			</div>
		</div>

		<div class="sbe-iff-img-container uk-margin-bottom">
			<div>
				<img ref="imagePicker" class="uk-width-1-1" v-bind:src="model.src" />
			</div>
		</div>

		<!-- <div>
				Coordinates:
				<input
					class="uk-input uk-width-1-1"
					type="text"
					readonly
					v-bind:value="coordinates"
				/>
		</div>-->

		<div class="uk-flex uk-flex-space-between">
			<div>
				Coordinate X:
				<input
					class="uk-input uk-width-expand"
					type="number"
					min="-1.0"
					max="1.0"
					step="0.05"
					v-model="model.x"
				/>
			</div>
			<div>
				Coordinate Y:
				<input
					class="uk-input uk-width-expand"
					type="number"
					min="-1.0"
					max="1.0"
					step="0.05"
					v-model="model.y"
				/>
			</div>
		</div>
	</div>
</template>

<script>
import { FocusPicker } from 'image-focus';

import { PLUGIN_NAME } from './const';

export default {
	mixins: [window.Storyblok.plugin],
	data() {
		return {
			picker: null,
			originalDimensions: '',
			coordinates: '',
			aspectRatio: '',
			mode: ''
		};
	},
	mounted() {
		this.setPicker();
		this.computeAspectRatio();
		window.addEventListener('resize', this.handleResize);
	},
	beforeDestroy() {
		window.removeEventListener('resize', this.handleResize);
	},
	methods: {
		handleResize() {
			this.updatePickerFocus({
				x: this.model.x || this.getDefaults().x,
				y: this.model.y || this.getDefaults().y
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
		computeAspectRatio() {
			// compute the image size
			const img = new Image();
			img.addEventListener('load', () => {
				// get the size
				const w = img.width;
				const h = img.height;
				// set the data
				this.originalDimensions = `${w}x${h}`;
				// if they are the same
				if (h === w) {
					this.aspectRatio = '1:1';
					this.mode = 'portrait';
				} else {
					// values for math
					let dividend = 0;
					let divisor = 0;
					// type of picture
					this.mode = 'unknown';
					// mode
					if (h > w) {
						dividend = h;
						divisor = w;
						this.mode = 'portrait';
					}
					if (w > h) {
						dividend = w;
						divisor = h;
						this.mode = 'landscape';
					}
					// get the gcd
					let remainder = 0;
					let gcd = -1;
					while (gcd === -1) {
						remainder = dividend % divisor;

						if (remainder === 0) {
							gcd = divisor;
						} else {
							dividend = divisor;
							divisor = remainder;
						}
					}

					// get the minimum common aspect ratio dividible
					let lowerWidth = w / gcd;
					let lowerHeight = h / gcd;

					// reduce them to smaller numbers
					let roundDown = false;
					const STEP = 10;
					const SUM_BOUND = 62;
					// reminders of the division
					// let reminderHeight = 0;
					// let reminderWidth = 0;
					while (lowerWidth + lowerHeight > SUM_BOUND) {
						// reminderHeight += lowerWidth - (lowerWidth /= STEP);
						// reminderWidth += lowerHeight - (lowerHeight /= STEP);
						lowerWidth /= STEP;
						lowerHeight /= STEP;
						roundDown = true;
					}

					let finalHeight = lowerHeight;
					let finalWidth = lowerWidth;
					if (roundDown) {
						// function to compute the delta associated with the width and height given
						const getDeltaObject = (original, width, height) => {
							const current = Math.abs((height + 1) / width);
							const delta = Math.abs(original - current);

							return {
								width,
								height,
								delta
							};
						};

						// original aspect ratio
						const original = h / w;
						// create objects
						const deltas = [];
						[0, 1].forEach(i => {
							[0, 1].forEach(j => {
								deltas.push(
									getDeltaObject(
										original,
										i ? Math.floor(lowerWidth) : Math.round(lowerWidth),
										j ? Math.floor(lowerHeight) : Math.round(lowerHeight)
									)
								);
							});
						});
						// reduce to the smallest one
						const smallest = deltas.reduce((pdo, cdo) =>
							cdo.delta < pdo.delta ? cdo : pdo
						);
						// reassign it back
						finalWidth = smallest.width;
						finalHeight = smallest.height;
					}

					// set the final aspect ratio
					this.aspectRatio = `${
						roundDown ? '~' : ''
					} ${finalWidth}:${finalHeight}`;
				}
			});
			img.src = this.model.src;
		},
		getDefaults() {
			return {
				plugin: PLUGIN_NAME,
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
		'model.x': {
			handler() {
				this.updatePickerFocus({
					x: this.model.x,
					y: this.model.y
				});
			}
		},
		'model.y': {
			handler() {
				this.updatePickerFocus({
					x: this.model.x,
					y: this.model.y
				});
			}
		},
		'model.src': {
			handler() {
				if (!this.model.src) {
					this.model.x = this.getDefaults().x;
					this.model.y = this.getDefaults().y;
				} else {
					this.computeAspectRatio();
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
.sbe-iff-img-container {
	max-width: 475px;
	margin-left: auto;
	margin-right: auto;
	border: 1px solid #e0e0e0;
}
</style>
