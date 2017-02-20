<template>
	<div id="data-range" v-show="range.length">
		<div class="labels" >
			<span
				v-for="(part, inx) in range"
				v-html="part.label"
				:style="styleLabel()"
				:class ='classLabel(part, inx)'
				:data-num="inx"
				></span>
		</div>
		<div class="scale">
			<span
				v-for="(part, inx) in range"
				:style="stylePeriod()"
				:class ='classPeriod(part, inx)'
				:data-num="inx"
				></span>
		</div>
		<input type="range" class="input"
			v-model="val"
			:style="styleInput()"
			:min="min"
			:max="max"
			:value="val"
			:step="step"
			@change= 'valChange()'
			>
	</div>
</template>

<script>

export default {
	props : {
		dataRange : {
			type: Array,
			default: function () {
				return [];
 			}
		},
		widthLabel : {
			type: Number,
			default : 40
		}
	},
	data () {
		return {
			widthSteep : 0,
			range : [],
			step : 1,
			min : 0,
			val : 0,
			max : 0
		}
	},
	methods : {
		/**
		 * Find first no empty element.
		 */
		valFindNoEmpty () {
			let count = 1;
			let that = this;
			let shift = that.val;
			let isNext = true;
			let len = that.range.length;

			while (isNext && count < len) {
				shift++;

				if (shift === len ) {
					shift = 0;
				}

				let next = that.range[shift];

				if (next && !next.isEmpty) {
					isNext = false;
					return that.val = shift;
				}

				count++;
			}
		},
		/**
		 * Hook after change value.
		 */
		valChangeAfterHook() {
			let that = this;

			if (window.vueEvents) {
				window.vueEvents.$emit('dataRangeChangeValue',  {
					val  : that.val,
					data : that.range[that.val]
				});
			}

		},
		/**
		 * Change value handel.
		 */
		valChange () {
			let that = this;

			let p = that.range[that.val];

			if (p.isEmpty) {
				that.valFindNoEmpty();
			}

			that.valChangeAfterHook();
		},
		/**
		 * Get class for label.
		 */
		classLabel (data, num) {
			let c = 'data-range-labels';

			if (this.val == num) {
				c += ' active';
			}

			return c;
		},
		/**
		 * Get class for period.
		 */
		classPeriod (data, num) {
			let c = 'period';

			if (data.isEmpty) {
				c += ' empty';
			}

			if (this.val == num) {
				c += ' active';
			}

			return c;
		},
		/**
		 * Return style for labels.
		 */
		styleLabel() {
			return `width: ${this.widthLabel}px;`
		},
		/**
		 * Return styke for input range.
		 */
		styleInput() {
			return `width: ${this.width}px;`
		},
		stylePeriod () {
			return `width: ${this.widthSteep}px;`
		}
	},
	created() {
		let that = this;
		if (that.dataRange.length) {
			that.range = that.dataRange;
		}
		that.val = 0;
		for (let i = 0; i < that.range.length; ++i) {
			let p = that.range[i];

			if (p.isFirst) {
				that.val = i;
				break;
			}
		}

		that.max = that.range.length - 1;

		if(that.max === -1) {
			that.max = 0;
		}

		if (that.range[that.val].isEmpty) {
			that.valFindNoEmpty();
		}

		if (window.vueEvents) {
			window.vueEvents.$on('cartogramsDataRangeUpdate', function () {
				if (that.range[that.val].isEmpty) {
					that.valFindNoEmpty();
				}

				that.$forceUpdate();
			});
		}

	},
	mounted () {
		let that = this;
		that.width = $('#data-range').width();
		that.widthSteep = that.width / (that.range.length - 1);
		let pos = $('#data-range > .scale').position();

		$('.data-range-labels').each(function() {
			let $el = $(this);
			let num = $el.data('num');

			$el.offset({
				top : pos.top - 40,
				left : ((pos.left - that.widthLabel/ 2) + that.widthSteep * num)
			});
		});

		$('#data-range > .scale > .period').each(function() {
			let $el = $(this);
			let num = $el.data('num');

			$el.offset({
				top : pos.top + 3,
				left : ((pos.left - that.widthLabel ) + that.widthSteep * num)
			});
		});

		that.$forceUpdate();
	}
}
</script>
