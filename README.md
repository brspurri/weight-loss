# Discovering ketosis:<br>How to effectively (and easily)<br> lose weight



*Here is a chart of my weight vs time in the past 16 months or so:*

![weight vs time in the past 16 months or so](weight.2015.png  "weight loss progress")

The chart was generated from the data file `weight.2015.csv` by the script `date-weight.r` in this git repository.  It requires `R` and `ggplot2`.





























In the following I'll describe the thought process, other people ideas,  and the software I used to lead me in the right direction.

## Salient facts & initial observations

- I used to be a pretty thin person. My 1st DMV card below, says 143 lb.
- Unfortunately, since moving to the US, I've been gaining more and
  more weight. I peaked in 2015, over 50 lbs higher.
- The US is a country where obesity is an epidemic.

![First DMV photo and weight (with full clothing)](1992-ariel-dmv.png "143 pounds, sometimes in the 90's")


Does a US typical lifestyle has anything to do with this epidemic? After reading on the subject, I could point at a few of the main suspects:

 - Fast food is the default food
 - Most food we buy and eat is heavily processed -- [Food, Inc. (documentary)](http://www.takepart.com/foodinc/film)
 - "No Fat" and "Low Fat" labels are everywhere on supermarket shelves
 - Many foods are enriched and sweetened with corn-syrup -- [Sugar Coated (documentary)](http://sugarcoateddoc.com/)

As in many other instances, I realized I need to think for myself. Ignore all "expert" advice. Question widely accepted ideas like the FDA "food pyramid". Start listening to my own body, my own logic & data I can collect myself and trust.

Once I did, the results followed.

## What didn't work

In the past, I tried several times to change my diet. After reading one of Atkins' books, I realized, checked, and accepted the fact that excess carbs are a major factor in gaining weight. But that realization alone has not led to success.

My will power, apparently, was insufficient. I had too much love of pizza and bread.  I would reduce my carb consumption, lose a few pounds (typically ~5 pounds), and then break-down, go back to consuming excess carbs, and gain all these pounds back, and then some. My longest diet stretch lasted just a few months.

It was obvious that something was missing in my method. I just had to find it. Obviously, I could increase my physical activity: say start training for a mini-marathon, but that's not what I felt comfortable with.

I realized early on that I need to adopt a lifesyle that not just reduces carbs, or add exercise, but is also sustainable and even enjoyable so it can turn into a painless routine. Something that:

> - I could do for years
> - Never feel the urge to break habits
> - Is not hard, or unpleasant for me to do


## Early insights & eureka moments

Early in the process I started using machine learning to identify the factors that make me gain or lose weight. I used a simple method: every morning I would weight myself and record both the new weights and whatever I did in the past ~24 hours, not just the food I ate, but also whether I exercised, slept too little or too much etc.

The format of the data collection file is very simple. A CSV with 3 columns:

> *Date*, *MorningWeight*, *Yesterday's lifestyle/food/actions*

The last column is a arbitrary-length list of *`word[:weight]`* items.

The (optional) numerical-weight following `:`, expresses higher/lower quantities. The default weight, when missing is 1:
 
    #
    # vim: textwidth=0 nowrap
    # Diet data file: 2012-06 and on
    #
    # 'sleep' is at least 8-hours of sleep.
    # Increase weights for more hours of sleep or larger quantities of food.
    #
    Date,MorningWeight,YesterdayFactors
    2012-06-10,185.0,
    2012-06-11,182.6,salad sleep bacon cheese tea halfnhalf icecream
    2012-06-12,181.0,sleep egg
    2012-06-13,183.6,mottsfruitsnack:2 pizza:5 bread:2 date:3 dietsnapple splenda milk nosleep
    2012-06-14,183.6,coffeecandy:5 egg mayo cheese:4 rice meat bread:0 peanut:5
    2012-06-15,183.4,meat sugarlesscandy salad cherry:4 bread:0 dietsnapple:2 egg mayo oliveoil
    2012-06-16,183.6,caprise bread grape:10 pasadena sugaryogurt dietsnapple:2 peanut:5 hotdog
    2012-06-17,182.6,grape:6 meat pistachio:5 peanut:5 cheese sorbet:5 orangejuice:2
    # and so on ...


Then I wrote [a script](./lifestyle-csv2vw) to convert this file to [vowpal-wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) training-set regression format. In the converted train-set the label is the change in weight (delta) in the past 24 hours, and the input features are what I've done or ate in the ~24 hours which led to this delta -- a straight copy of the 3rd column.

I was not dieting at that time. Just collecting data.

The machine learning process error convergence after partly sorting
the lines descending, by delta (to reduce noise) looks like this:

![error convergence (after partial descending sort by delta)](vw-convergence.png  "loss convergence in 4 data passes")

You can reproduce my work by building your own data-file, installing vowpal-wabbit, and its utility [`vw-varinfo`](https://github.com/JohnLangford/vowpal_wabbit/wiki/using-vw-varinfo), and running `make` in this directory.

Here's how a typical result of running `make` looks like.

The positive relative-score values are life-style choices that make you gain weight, while the negative ones make you lose weight. This particular data set is very noisy, since:

- The number of examples is too small
- My scales have a large error
- Items that make you both lose and gain weight, often appear on the same line so they cancel each other and confuse the learning process.

So I focused mostly on the extremes (start and end) of the list which looked like this:

    FeatureName       HashVal   ...   Weight RelScore
    nosleep            143407   ...  +0.6654 90.29%
    watermelon         149311   ...  +0.4636 62.91%
    sugarlemonade      203375   ...  +0.3975 53.94%
    trailmix           174671   ...  +0.3362 45.63%
    bread              135055   ...  +0.3345 45.40%
    caramelizedwalnut  148079   ...  +0.3316 44.99%
    bun                  1791   ...  +0.3094 41.98%

    ... (trimmed for brevity) ...

    stayhome           148879   ...  -0.2690 -36.50%
    bacon               64431   ...  -0.2998 -40.69%
    egg                197743   ...  -0.3221 -43.70%
    parmagian          121679   ...  -0.3385 -45.94%
    oliveoil           156831   ...  -0.3754 -50.95%
    halfnhalf          171855   ...  -0.4673 -63.41%
    sleep              127071   ...  -0.7369 -100.00%

Despite the noisy & insufficient data, and the inaccuracies in weighting, the machine-learning experiments made 4 facts very obvious, pretty early:

- Sleeping longer consistently appeared as #1 factor in losing weight.
- Lack of sleep did the opposite: too little sleep lead to weight gains.
- Carbs make you gain weight. The worst are high-starch and sugary foods.
- Fatty and oily foods do the opposite: they make you lose weight.

The 'stayhome' lifestlye, which fell mostly on weekends, was of a red-herring, I simply slept longer when I didn't have to commute to work.

It took me a while to figure out the sleep part. When we sleep we don't eat. It is that simple.

Moreover: we tend to binge and snack while not particularly hungry, but we never do it during sleep.

And our sleeping time is our fasting time.

But it is not enough to fast. We also need to channel our body chemistry to break-up excess stored-fat whenever we fast.


## Further progress

You may note that in the top chart there's a notable acceleration in the rate of weight loss.  The cause was deeper insights and better ability to sustain the diet the more I understood the causes of weight gain and loss.

Extending the fasting time was one major accelerator of weight-loss rate. I did that by:

- Skipping breakfast and
- Stop eating earlier in the evening before going to bed.

This gave me 14-16 hours of fasting each day. Rather than the more typical 10-12 hours/day of fasting.

The 2nd accelerator was loading up on fat in order to feel full.

I now feel pretty confident that I can go all the way back to my original weight when I first landed on US soil.

At the present rate, it should take not more than 2 years to completely reverse the damage of the past 20 years. 

It is important to stress that I also feel much better the more weight I lose. As a welcome side-effect, all the bordeline-high levels in my blood tests, have moved significantly towards normal averages during the same period when I lost weight.

Beware of doctors [who push statins](https://www.google.com/search?q=the+truth+about+statins) instead of suggesting a better diet. Doubt anyone who tells you you need to reduce fat. Run away if they tell you "high cholesterol" is dangerous.

My data has taught me that the body is an amazing machine. It is extremely adaptive. Cholesterol is an essential building block and your liver produces as much as it needs of it. It is not our ***high fat consumption***, it is the ***storage of fat process*** that makes us unhealthy.  An enzyme called Lipase breaks-up fat. To raise the levels of Lipase so our body fat gets converted to energy, we need to give the body fat as an alternative to carbohydrates. When the body has no blood sugar, and no glycogen left in the liver, it has no other choice but to adapt and compensate.  The source of energy (ATP synthesis) is switched from carbs to fats by producing more fat-breaking agents.

When Lipase (and all other enzymes in the fat-to-ATP chemical path) mobilize and their levels become elevated, we reach a new steady state (called ketosis) and that's when we start winning this fight.

In ketosis, our night sleep (fasting time) becomes our best friend.  The fat-breaking agents keep working while we sleep, breaking-up the stored fat. This leads to weight-loss, and a healthier body.


## The bottom-line recipe - all you need to succeed:

- The hardest part (especially at the beginning) is reducing carbs. The worst carbs are starches (pizza, pasta, bread etc.), then processed foods with high sugar content (sweet sodas, other juices, etc). This doesn't mean ***no*** carbs. You may afford yourself carbs from time to time (say a pizza once a week). As it turns out, an occasional lapse isn't enough to get you out of ketosis. However, you need to make sure you consume ***much less carbs*** and ***less frequently*** than before. In particular, you must avoid binges of snacks like chips, pizza, doughnuts, pasta, and bread.

- [Look-up Glycemic index on wikipedia](https://en.wikipedia.org/wiki/Glycemic_index). Avoid foods with high glycemic index. This prevents the blood sugar spikes which lead to insulin spikes and tell the body chemical cycles to revert back from ketosis to fat-accumulation.  Have a sweet tooth? Eat an orange instead of drinking orange juice. The two have vastly different glycemic indexes and this makes a huge difference. If you must add sweetness to your cup of tea or coffee, use a Splenda (sucralose) tablet (~0.1 gram) rather than a tea-spoon of sugar (several grams). Result: same sweetness effect, but much lower blood-glucose.

- High fat: switch from milk to half-and-half. It has less carbs (lactose) and more fat; plus, it tastes better.  Eat avocados, olive oil, mayo, coconut oil, nuts.  Never worry about *natural* fat, you can eat as much as you want of it. This is what makes it much easier to avoid carbs. When you stuff yourself with fat you feel much less hungry and miss the carbs less. The beautiful thing is that the body is very good at figuring out "I have too much fat in the blood, so let's increase the amount of enzymes which break-up fat" and this makes you lose weight in the long run.  Most importantly, avoid products labeled "low fat" or "fat free". The food industry usually replaces fat with sugar (so it tastes better, otherwise it tastes awful). You'll often hear about bad vs good fat. My take: as long as it is natural, it is ok. Trans-fat is relatively rare in nature, most of it is artificially hydrogenated by the food industry. The less saturated fat is, the better. Mono-saturated (plant) fat is the best, then come the poly-unsaturated fats, and finally near saturated (but not fully saturated) fats that come from animals. The buttery-spread spectrum is:  *Margarine: no; Butter: ok; Earth Balance: no problem*.

- A bit of exercise.  Of course more is better, but for many this may prove difficult. I don't excercise too much. I just bike to work and back about 20 min each way, meaning 40 min/day, 5 out of 7 days/week. Walking the dog (but walk faster), or Zumba dance to music. The trick is to find something that you don't find hard to do. Or find company to do it together. Then do a little bit of it every day.

- Longer fasting periods: sleep longer, stop eating as early as possible before going to sleep and start eating as late as possible after sleeping. Skip breakfast, after some time you won't feel hungry in the morning anymore.  After long periods of fasting, the body chemistry adjusts: I need more ATP, but I don't have glucose in the blood, and the glycogen in the liver is all fully consumed (takes about 1-2 days of low or no carbs) so there's no other option but to start and break-up stored fat. This elevates the enzymes that help with breaking up fat and the Krebs cycle reverses direction in the paths where it matters.

- Eat eggs.  They are a wonderful combo of fat and protein with no carbs at all.  I read an interview with a Japanese woman who reached 124 years and one of her secrets was to eat an egg daily.  My favorite food is a scrambled egg with grilled onions (onions are a bit high on carbs, but too tasty to give up) and olives.

- Eat slower, chew longer... don't swallow just yet! Humans, just like dogs, tend to swallow too soon. Stop eating when you feel full. This comes from another recent interview with a 102 year-old man listing his secrets.

***

## Further reading:

- [The Krebs (aka Citric acid) cycle](https://en.wikipedia.org/wiki/Citric_acid_cycle)
- [Spikes of Insulin and their effects](https://en.wikipedia.org/wiki/Sugar_crash) - what the body does when it has excess of sugar vs excess of fat.
- [Glycemic Index](https://en.wikipedia.org/wiki/Glycemic_index)
- [Glycogen and its storage in the liver](https://en.wikipedia.org/wiki/Glycogen)


And watch this nice 7:41 minute video of James McCarter in Quantified Self (an eye opener for me):

- [James McCarter: The Effects of a Year in Ketosis](https://vimeo.com/147795263) 



