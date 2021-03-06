<p>I’m sure you’ve seen the <a href="https://github.com/blog/817-behold-image-view-modes">image view modes</a> Github released last month. It’s a really nice way to see the differences between two versions of an image. In this article, I’ll try to explain how a simple image diff could be built using pure Ruby and <a href="https://github.com/wvanbergen/chunky_png">ChunkyPNG</a>.</p>
<p>If you need a more basic introduction to working with pixel data in ChunkyPNG, check out <a href="http://jeffkreeftmeijer.com/2011/pure-ruby-colored-blob-detection">last week’s article</a>, which I did some simple blob detection.</p>
<p>In its simplest form, finding differences in images works by looping over each pixel in the first image and checking if it’s the same as the pixel in the same spot in the second image. An implementation might look like this:</p>
<div class="highlight">
<pre><code class="ruby"><span class="nb">require</span> <span class="s1">'chunky_png'</span>

<span class="n">images</span> <span class="o">=</span> <span class="o">[</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'1.png'</span><span class="p">),</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'2.png'</span><span class="p">)</span>
<span class="o">]</span>

<span class="n">diff</span> <span class="o">=</span> <span class="o">[]</span>

<span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">height</span><span class="o">.</span><span class="n">times</span> <span class="k">do</span> <span class="o">|</span><span class="n">y</span><span class="o">|</span>
  <span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">row</span><span class="p">(</span><span class="n">y</span><span class="p">)</span><span class="o">.</span><span class="n">each_with_index</span> <span class="k">do</span> <span class="o">|</span><span class="n">pixel</span><span class="p">,</span> <span class="n">x</span><span class="o">|</span>
    <span class="n">diff</span> <span class="o">&lt;&lt;</span> <span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span> <span class="k">unless</span> <span class="n">pixel</span> <span class="o">==</span> <span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span>
  <span class="k">end</span>
<span class="k">end</span>

<span class="nb">puts</span> <span class="s2">"pixels (total):     </span><span class="si">#{</span><span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">pixels</span><span class="o">.</span><span class="n">length</span><span class="si">}</span><span class="s2">"</span>
<span class="nb">puts</span> <span class="s2">"pixels changed:     </span><span class="si">#{</span><span class="n">diff</span><span class="o">.</span><span class="n">length</span><span class="si">}</span><span class="s2">"</span>
<span class="nb">puts</span> <span class="s2">"pixels changed (%): </span><span class="si">#{</span><span class="p">(</span><span class="n">diff</span><span class="o">.</span><span class="n">length</span><span class="o">.</span><span class="n">to_f</span> <span class="o">/</span> <span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">pixels</span><span class="o">.</span><span class="n">length</span><span class="p">)</span> <span class="o">*</span> <span class="mi">100</span><span class="si">}</span><span class="s2">%"</span>

<span class="n">x</span><span class="p">,</span> <span class="n">y</span> <span class="o">=</span> <span class="n">diff</span><span class="o">.</span><span class="n">map</span><span class="p">{</span> <span class="o">|</span><span class="n">xy</span><span class="o">|</span> <span class="n">xy</span><span class="o">[</span><span class="mi">0</span><span class="o">]</span> <span class="p">},</span> <span class="n">diff</span><span class="o">.</span><span class="n">map</span><span class="p">{</span> <span class="o">|</span><span class="n">xy</span><span class="o">|</span> <span class="n">xy</span><span class="o">[</span><span class="mi">1</span><span class="o">]</span> <span class="p">}</span>

<span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">.</span><span class="n">rect</span><span class="p">(</span><span class="n">x</span><span class="o">.</span><span class="n">min</span><span class="p">,</span> <span class="n">y</span><span class="o">.</span><span class="n">min</span><span class="p">,</span> <span class="n">x</span><span class="o">.</span><span class="n">max</span><span class="p">,</span> <span class="n">y</span><span class="o">.</span><span class="n">max</span><span class="p">,</span> <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Color</span><span class="o">.</span><span class="n">rgb</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="mi">255</span><span class="p">,</span><span class="mi">0</span><span class="p">))</span>
<span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="s1">'diff.png'</span><span class="p">)</span>
</code></pre>
</div>
<p><span class="small">Want the code? Here’s a <a href="https://gist.github.com/923894">Gist</a>.</span></p>
<p>After loading in the two images, we’ll loop over the pixels of the first one. If the pixel is the same as the one in the second image, we’ll add it to the <code>diff</code> array. When we’re done, we’ll draw a bounding box around the area that contains the changes:</p>
<p><img src="http://jeffkreeftmeijer.com/images/tapir.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_hat.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_diff_box.png" style="width:130px; margin:0 5px;"></p>
<p>It worked! The result image has a bounding box around the hat we added to the image and the output tells us that almost 9% of the pixels in the image changed, which seems about right.</p>
<pre><code>pixels (total):     16900
pixels changed:     1502
pixels changed (%): 8.887573964497042%</code></pre>
<p>A problem with this approach is that it only <em>detects</em> change, without <em>measuring</em> it. It doesn’t care if the pixel it’s looking at is just a bit darker or a completely different color. If we use this code to compare one image to a slightly darker version of itself, the result will look like this:</p>
<p><img src="http://jeffkreeftmeijer.com/images/tapir_hat.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_hat_dark.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_diff_dark_box.png" style="width:130px; margin:0 5px;"></p>
<pre><code>pixels (total):     16900
pixels changed:     16900
pixels changed (%): 100.0%</code></pre>
<p>This would mean that the two images are completely different, while (from a human eye’s perspective) they’re almost the same. To get a more accurate result, we’ll need to measure the difference in the pixels’ colors.</p>
<h3>Calculating color difference</h3>
<p>To calculate the color difference, we’ll use the the <a href="http://en.wikipedia.org/wiki/Color_difference">ΔE*</a> (“Delta E”) distance metric. There are a couple of different versions of this metric, but we’ll take the first one (<a href="http://en.wikipedia.org/wiki/Color_difference#CIE76">CIE76</a>), since it’s the simplest and we don’t need anything too fancy. The ΔE* metric was created for the <a href="http://en.wikipedia.org/wiki/Lab_color_space"><span class="caps">LAB</span> color space</a>, which was designed to approximate human vision. In this example, we’re not going to worry about converting to <span class="caps">LAB</span>, so we’ll just use the <span class="caps">RGB</span> color space (note that this will mean our results will be less accurate). If you want to know more about the difference, check out <a href="http://stevehanov.ca/blog/index.php?id=116">this demo</a>.</p>
<p>Again, we loop over every pixel in the images. If they’re different, we calculate how different they are using the ΔE* metric and store that in the <code>diff</code> array. We also use that score to calculate a grayscale color value we use on the result image:</p>
<div class="highlight">
<pre><code class="ruby"><span class="nb">require</span> <span class="s1">'chunky_png'</span>
<span class="kp">include</span> <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Color</span>


<span class="n">images</span> <span class="o">=</span> <span class="o">[</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'1.png'</span><span class="p">),</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'2.png'</span><span class="p">)</span>
<span class="o">]</span>

<span class="n">output</span> <span class="o">=</span> <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">new</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">width</span><span class="p">,</span> <span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">.</span><span class="n">width</span><span class="p">,</span> <span class="no">WHITE</span><span class="p">)</span>

<span class="n">diff</span> <span class="o">=</span> <span class="o">[]</span>

<span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">height</span><span class="o">.</span><span class="n">times</span> <span class="k">do</span> <span class="o">|</span><span class="n">y</span><span class="o">|</span>
  <span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">row</span><span class="p">(</span><span class="n">y</span><span class="p">)</span><span class="o">.</span><span class="n">each_with_index</span> <span class="k">do</span> <span class="o">|</span><span class="n">pixel</span><span class="p">,</span> <span class="n">x</span><span class="o">|</span>
    <span class="k">unless</span> <span class="n">pixel</span> <span class="o">==</span> <span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span>
      <span class="n">score</span> <span class="o">=</span> <span class="no">Math</span><span class="o">.</span><span class="n">sqrt</span><span class="p">(</span>
        <span class="p">(</span><span class="n">r</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="n">r</span><span class="p">(</span><span class="n">pixel</span><span class="p">))</span> <span class="o">**</span> <span class="mi">2</span> <span class="o">+</span>
        <span class="p">(</span><span class="n">g</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="n">g</span><span class="p">(</span><span class="n">pixel</span><span class="p">))</span> <span class="o">**</span> <span class="mi">2</span> <span class="o">+</span>
        <span class="p">(</span><span class="n">b</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="n">b</span><span class="p">(</span><span class="n">pixel</span><span class="p">))</span> <span class="o">**</span> <span class="mi">2</span>
      <span class="p">)</span> <span class="o">/</span> <span class="no">Math</span><span class="o">.</span><span class="n">sqrt</span><span class="p">(</span><span class="no">MAX</span> <span class="o">**</span> <span class="mi">2</span> <span class="o">*</span> <span class="mi">3</span><span class="p">)</span>

      <span class="n">output</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span> <span class="o">=</span> <span class="n">grayscale</span><span class="p">(</span><span class="no">MAX</span> <span class="o">-</span> <span class="p">(</span><span class="n">score</span> <span class="o">*</span> <span class="no">MAX</span><span class="p">)</span><span class="o">.</span><span class="n">round</span><span class="p">)</span>
      <span class="n">diff</span> <span class="o">&lt;&lt;</span> <span class="n">score</span>
    <span class="k">end</span>
  <span class="k">end</span>
<span class="k">end</span>

<span class="nb">puts</span> <span class="s2">"pixels (total):     </span><span class="si">#{</span><span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">pixels</span><span class="o">.</span><span class="n">length</span><span class="si">}</span><span class="s2">"</span>
<span class="nb">puts</span> <span class="s2">"pixels changed:     </span><span class="si">#{</span><span class="n">diff</span><span class="o">.</span><span class="n">length</span><span class="si">}</span><span class="s2">"</span>
<span class="nb">puts</span> <span class="s2">"image changed (%): </span><span class="si">#{</span><span class="p">(</span><span class="n">diff</span><span class="o">.</span><span class="n">inject</span> <span class="p">{</span><span class="o">|</span><span class="n">sum</span><span class="p">,</span> <span class="n">value</span><span class="o">|</span> <span class="n">sum</span> <span class="o">+</span> <span class="n">value</span><span class="si">}</span><span class="s2"> / images.first.pixels.length) * 100}%"</span>

<span class="n">output</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="s1">'diff.png'</span><span class="p">)</span>
</code></pre>
</div>
<p><span class="small">Want the code? Here’s a <a href="https://gist.github.com/925244">Gist</a>.</span></p>
<p>Now we have a more accurate difference score. If we look at the output, we can see that less than 3% of the image was changed:</p>
<pre><code>pixels (total):    16900
pixels changed:    1502
image changed (%): 2.882157784948056%</code></pre>
<p>Again, a diff image is saved. This time, it shows the differences using shades of gray. Bigger changes are darker:</p>
<p><img src="http://jeffkreeftmeijer.com/images/tapir.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_hat.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_diff_delta_e.png" style="width:130px; margin:0 5px;"></p>
<p>Now, let’s try the two images where the second one is slightly darker:</p>
<pre><code>pixels (total):    16900
pixels changed:    16900
image changed (%): 5.4418255392228945%</code></pre>
<p><img src="http://jeffkreeftmeijer.com/images/tapir_hat.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_hat_dark.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_diff_delta_e_dark.png" style="width:130px; margin:0 5px;"></p>
<p>Great. Now our code knows that the images are only darker, not completely different. If you look closely, you can see the difference in the result image.</p>
<h3>What about Github?</h3>
<p>Github uses a <a href="http://en.wikipedia.org/wiki/Blend_modes#Difference">difference blend</a>, which might be familiar if you’ve worked with image-editing software like Photoshop before. Doing something like that is quite simple. We loop over every pixel in the two images and calculate their difference per <span class="caps">RGB</span> channel:</p>
<div class="highlight">
<pre><code class="ruby"><span class="nb">require</span> <span class="s1">'chunky_png'</span>
<span class="kp">include</span> <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Color</span>

<span class="n">images</span> <span class="o">=</span> <span class="o">[</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'1.png'</span><span class="p">),</span>
  <span class="ss">ChunkyPNG</span><span class="p">:</span><span class="ss">:Image</span><span class="o">.</span><span class="n">from_file</span><span class="p">(</span><span class="s1">'2.png'</span><span class="p">)</span>
<span class="o">]</span>

<span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">height</span><span class="o">.</span><span class="n">times</span> <span class="k">do</span> <span class="o">|</span><span class="n">y</span><span class="o">|</span>
  <span class="n">images</span><span class="o">.</span><span class="n">first</span><span class="o">.</span><span class="n">row</span><span class="p">(</span><span class="n">y</span><span class="p">)</span><span class="o">.</span><span class="n">each_with_index</span> <span class="k">do</span> <span class="o">|</span><span class="n">pixel</span><span class="p">,</span> <span class="n">x</span><span class="o">|</span>

    <span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span> <span class="o">=</span> <span class="n">rgb</span><span class="p">(</span>
      <span class="n">r</span><span class="p">(</span><span class="n">pixel</span><span class="p">)</span> <span class="o">+</span> <span class="n">r</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="mi">2</span> <span class="o">*</span> <span class="o">[</span><span class="n">r</span><span class="p">(</span><span class="n">pixel</span><span class="p">),</span> <span class="n">r</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span><span class="o">].</span><span class="n">min</span><span class="p">,</span>
      <span class="n">g</span><span class="p">(</span><span class="n">pixel</span><span class="p">)</span> <span class="o">+</span> <span class="n">g</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="mi">2</span> <span class="o">*</span> <span class="o">[</span><span class="n">g</span><span class="p">(</span><span class="n">pixel</span><span class="p">),</span> <span class="n">g</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span><span class="o">].</span><span class="n">min</span><span class="p">,</span>
      <span class="n">b</span><span class="p">(</span><span class="n">pixel</span><span class="p">)</span> <span class="o">+</span> <span class="n">b</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span> <span class="o">-</span> <span class="mi">2</span> <span class="o">*</span> <span class="o">[</span><span class="n">b</span><span class="p">(</span><span class="n">pixel</span><span class="p">),</span> <span class="n">b</span><span class="p">(</span><span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">[</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="o">]</span><span class="p">)</span><span class="o">].</span><span class="n">min</span>
    <span class="p">)</span>
  <span class="k">end</span>

<span class="k">end</span>

<span class="n">images</span><span class="o">.</span><span class="n">last</span><span class="o">.</span><span class="n">save</span><span class="p">(</span><span class="s1">'diff.png'</span><span class="p">)</span>
</code></pre>
</div>
<p><span class="small">Want the code? Here’s a <a href="https://gist.github.com/924996">Gist</a>.</span></p>
<p>Using that, comparing the two images to the left would result in the diff-image on the right, nicely showing what changed:</p>
<p><img src="http://jeffkreeftmeijer.com/images/tapir.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_hat.png" style="width:130px; margin:0 5px;"><img src="http://jeffkreeftmeijer.com/images/tapir_diff_blend.png" style="width:130px; margin:0 5px;"></p>
<p>Because the colors are compared by channel (R,G and B) instead of as one color, three scores are returned. This means the output image is in color, but comparing the channels separately can make the result less accurate.</p>
<p>As always, if you used this idea to build something yourself, know of a way to improve the code or have some questions or tips, be sure to let me know. If you want to know more about something I talked about, be sure to suggest it as a next article.</p>