<style media="screen">
	#data-range {
		width : 600px;
	}
	#data-range > .labels, #data-range > .labels > span {
		position: absolute;
	}
</style>
<template>
	<div id="data-range">
		<div class="labels" >
			<span
				v-for="(part, inx) in range"
				v-html="part.label"
				:style="styleLabel()"
				:data-num="inx"
				:class ='classLabel(part, inx)'
				></span>
		</div>
		<input type="range" class="input"
			v-model="val"
			:style="styleInput()"
			:min="min"
			:max="max"
			:step="step"
			@change= 'valChange()'
			>
	</div>
</template>

<script>
let range = [
	{
		label : '2015<br/>Весна',
		val : 1,
		isEmpty : true
	},
	{
		label : '2015<br/>Лето',
		val : 2,
		isEmpty : true
	},
	{
		label : '2015<br/>Осень',
		val : 3,
		isEmpty : true
	},
	{
		label : '2015<br/>3има',
		val : 4,
		isFirst : true
	},
	{
		label : '2016<br/>3има',
		val : 5
	},
	{
		label : '2016<br/>Весна',
		val : 6
	},
	{
		label : '2016<br/>Лето',
		val : 7
	},
	{
		label : '2016<br/>Осень',
		val : 8
	},
	{
		label : '2016<br/>Зима',
		val : 9
	},
	{
		label : '2017<br/>Зима',
		val : 10
	}
];
export default {
	props : {
		data : {
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
			that.range[that.val];
			 console.log('Hook tell', {
				 val  : that.val,
				 data : that.range[that.val]
			 });
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
		}

	},
	created() {
		let that = this;
		that.range = range;

		for (let i = 0; i < that.range.length; ++i) {
			let p = that.range[i];

			if (p.isFirst) {
				that.val = i;
				break;
			}
		}
		that.val = 0;
		that.max = that.range.length - 1;

		if(that.max === -1) {
			that.max = 0;
		}

		that.valFindNoEmpty();
	},
	mounted () {
		let that = this;
		let $cont = $('#data-range');
		let pos = $('#data-range > .input').position();
		that.width = $cont.width();

		$('.data-range-labels').each(function() {
			let $el = $(this);
			let num = $el.data('num');
			let elWidth = 0;

			$el.offset({
				top : pos.top - 40,
				left : ((pos.left - 20) + (that.width / (that.range.length - 1) ) * num)
			});
		});
	}
}
</script>
