# In-class-assigment
<img width="1920" height="1080" alt="Screenshot (173)" src="https://github.com/user-attachments/assets/125590e4-1c2b-43da-a0d8-b3ada36d1409" />
<img width="1920" height="1080" alt="Screenshot (172)" src="https://github.com/user-attachments/assets/408fd6a0-d6e6-4652-a318-3f5313c6925a" />
<img width="1920" height="1080" alt="Screenshot (171)" src="https://github.com/user-attachments/assets/befa3d5b-eee9-4c7b-94ec-a936fed0c90b" />
<img width="1920" height="1080" alt="Screenshot (170)" src="https://github.com/user-attachments/assets/dfea98bc-6bfa-4c30-9a6c-f6e290c8bd94" />
<img width="1920" height="1080" alt="Screenshot (169)" src="https://github.com/user-attachments/assets/6d410920-c378-4600-a105-2377fcc3eec8" />
<img width="1920" height="1080" alt="Screenshot (168)" src="https://github.com/user-attachments/assets/608a565a-0b2e-41ab-88dc-ea96c378bb57" />

%% In-class exercises: image processing basics
% Leslie Foncham Mbunda, Savonia UAS, Internet of Things
%
% Images used (built-in MATLAB / Image Processing Toolbox images, not uploaded):
% cameraman.tif, pout.tif, coins.png
clear; close all; clc;
rng(0);   % same random noise on every run, so the numbers are reproducible

%% Exercise 1: pixels and quantization
% Load cameraman.tif and print size and class
I = imread('cameraman.tif');
fprintf('Size: %d x %d, class: %s\n', size(I,1), size(I,2), class(I));

% Quantize to 4, 2 and 1 bit (2^k levels, stretched back to 0..255 for display)
bits = [4 2 1];
Qimg = cell(1, 3);
for n = 1:3
    levels = 2^bits(n);
    step   = 256 / levels;
    Qimg{n} = uint8(floor(double(I) / step) * 255 / (levels - 1));
end
figure;
montage({I, Qimg{:}}, 'Size', [1 4]);
title('8 bit | 4 bit | 2 bit | 1 bit');

% Downsample by 8 with and without imresize
A1 = I(1:8:end, 1:8:end);     % without imresize: keep every 8th pixel
A2 = imresize(I, 1/8);        % with imresize: antialiasing + bicubic
fprintf('Downsampled sizes: %dx%d (manual), %dx%d (imresize)\n', size(A1), size(A2));
figure;
montage({I, imresize(A1, 8, 'nearest'), imresize(A2, 8, 'nearest')}, 'Size', [1 3]);
title('Original | Every 8th pixel | imresize 1/8 (both enlarged x8)');

%% Exercise 1: think about
% *Bit depth:* 4 bit looks almost identical to the original, 2 bit shows clear
% false contours in the sky and on the coat, and at 1 bit only a black-and-white
% silhouette is left. The scene is still recognizable at 1 bit, but fine detail
% is gone from 2 bit downward.
%
% *Aliasing:* it appears in the version made without imresize. Thin structures
% such as the tripod legs and the camera break into jagged, broken lines, and
% the grass texture turns into random blocks, because no low-pass filtering was
% done before throwing pixels away. imresize smooths first, so it looks blurry
% but not jagged.

%% Exercise 2: degrade an image
Id = im2double(I);

% Gaussian noise (variance 0.01) and salt & pepper noise (density 0.05)
N_gauss = imnoise(I, 'gaussian', 0, 0.01);
N_sp    = imnoise(I, 'salt & pepper', 0.05);

% Motion blur
h_motion = fspecial('motion', 15, 30);
B = imfilter(I, h_motion, 'replicate');

% Brightness ramp multiplied onto the image
[rows, cols] = size(Id);
ramp = repmat(linspace(0, 1, cols), rows, 1);
L = Id .* ramp;

figure;
montage({N_gauss, N_sp, B, L}, 'Size', [1 4]);
title('Gaussian noise | Salt & pepper | Motion blur | Brightness ramp');

%% Exercise 2: think about
% *Additive vs multiplicative:* Gaussian noise is additive (a random value is
% added to every pixel). The brightness ramp is multiplicative (each pixel is
% multiplied by a factor between 0 and 1). Salt & pepper is neither: it replaces
% pixels with 0 or 255. Motion blur is a convolution (a weighted sum of
% neighbouring pixels).
%
% *Median filter:* it fixes salt & pepper noise, because the black and white
% dots are extreme values and never become the median of a neighbourhood.

%% Exercise 3: contrast and geometry
P = imread('pout.tif');

% Histogram
figure;
subplot(1,2,1); imshow(P); title('pout.tif');
subplot(1,2,2); imhist(P); title('Histogram');

% imadjust, gamma 0.5, gamma 2 and histeq
imgs   = {P, imadjust(P), imadjust(P, [], [], 0.5), imadjust(P, [], [], 2), histeq(P)};
titles = {'Original', 'imadjust', 'Gamma 0.5', 'Gamma 2', 'histeq'};
figure;
for k = 1:5
    subplot(2,5,k);   imshow(imgs{k}); title(titles{k});
    subplot(2,5,k+5); imhist(imgs{k});
end

% Enlarge a 40x40 patch with nearest and bicubic
patch = P(60:99, 100:139);
figure;
subplot(1,3,1); imshow(patch);                           title('Patch 40x40');
subplot(1,3,2); imshow(imresize(patch, 8, 'nearest'));   title('Nearest x8');
subplot(1,3,3); imshow(imresize(patch, 8, 'bicubic'));   title('Bicubic x8');

%% Exercise 3: think about
% *Best method:* imadjust looks the most natural. The original histogram is
% squeezed into a narrow middle band, and imadjust stretches it to the full
% 0-255 range. histeq gives even more contrast but looks harsh and grainy.
% Gamma alone does not fix the low contrast; it only makes the image lighter
% or darker.
%
% *Why gamma < 1 brightens:* the output is in^gamma with values between 0 and 1.
% A power below 1 makes such numbers larger (0.25^0.5 = 0.5), so dark tones are
% lifted the most.
%
% *Bicubic vs nearest:* nearest copies the closest pixel, so the patch turns into
% blocks. Bicubic computes new in-between values from a 4x4 neighbourhood with
% a smooth cubic weighting, so gradients and edges look smooth.

%% Exercise 4: filter and compare
noisy  = {N_gauss, N_sp};
nNames = {'Gaussian noise', 'Salt & pepper'};
fNames = {'Mean', 'Gaussian', 'Median'};
h_mean = fspecial('average', 3);

results = cell(2, 3);
psnrVals = zeros(2, 4);
for n = 1:2
    X = noisy{n};
    results{n,1} = imfilter(X, h_mean, 'replicate');
    results{n,2} = imgaussfilt(X, 1);
    results{n,3} = medfilt2(X, [3 3]);
    psnrVals(n,1) = psnr(X, I);
    for f = 1:3
        psnrVals(n,f+1) = psnr(results{n,f}, I);
    end
end

figure;
for n = 1:2
    subplot(2,4,(n-1)*4+1); imshow(noisy{n}); title(nNames{n});
    for f = 1:3
        subplot(2,4,(n-1)*4+1+f); imshow(results{n,f}); title(fNames{f});
    end
end

% PSNR table (dB, higher is better)
PSNR_table = array2table(round(psnrVals, 2), ...
    'VariableNames', {'Noisy', 'Mean', 'Gaussian', 'Median'}, ...
    'RowNames', nNames)

% Spectrum of the original and of the motion-blurred image
S_orig = log(1 + abs(fftshift(fft2(im2double(I)))));
S_blur = log(1 + abs(fftshift(fft2(im2double(B)))));
figure;
subplot(2,2,1); imshow(I);          title('Original');
subplot(2,2,2); imshow(S_orig, []); title('Spectrum: original');
subplot(2,2,3); imshow(B);          title('Motion blurred');
subplot(2,2,4); imshow(S_blur, []); title('Spectrum: blurred');

%% Exercise 4: think about
% *Which filter wins:* see the PSNR table. For salt & pepper noise the median
% filter wins clearly, because it removes the isolated dots completely. For
% Gaussian noise the mean and Gaussian filters do best, since averaging is what
% reduces random additive noise.
%
% *Spectrum after blurring:* the high frequencies (fine detail, far from the
% centre) are strongly reduced, so the spectrum gets darker towards the edges.
% Motion blur also adds parallel dark lines perpendicular to the 30 degree blur
% direction.

%% Exercise 5: edges
% Sobel and Canny
E_sobel = edge(I, 'sobel');
E_canny = edge(I, 'canny');
figure;
montage({I, E_sobel, E_canny}, 'Size', [1 3]);
title('Original | Sobel | Canny');

% Canny with sigma 1, 2 and 3
sigmas = [1 2 3];
E_sig = cell(1, 3);
for k = 1:3
    E_sig{k} = edge(I, 'canny', [], sigmas(k));
    fprintf('Canny sigma = %d: %d edge pixels\n', sigmas(k), nnz(E_sig{k}));
end
figure;
montage(E_sig, 'Size', [1 3]);
title('Canny \sigma = 1 | \sigma = 2 | \sigma = 3');

% Canny on the noisy image with and without smoothing
E_noisy    = edge(N_gauss, 'canny', [], 0.5);           % (almost) no smoothing
E_smoothed = edge(imgaussfilt(N_gauss, 2), 'canny');    % smoothed first
nNoisy = nnz(E_noisy);
nSmooth = nnz(E_smoothed);
fprintf('Noisy image, no smoothing: %d edge pixels\n', nNoisy);
fprintf('Noisy image, smoothed:     %d edge pixels\n', nSmooth);
fprintf('Edge pixels removed by smoothing: %d\n', nNoisy - nSmooth);
figure;
montage({E_noisy, E_smoothed}, 'Size', [1 2]);
title('Canny on noisy image: no smoothing | smoothed');

%% Exercise 5: think about
% *Effect of sigma:* a larger sigma smooths more before edge detection, so the
% edge map keeps only the main outlines (man, camera, horizon) and the number of
% edge pixels drops. Small sigma keeps fine texture such as the grass, but also
% more noise.
%
% *False edges removed:* the difference in nnz printed above. Without smoothing,
% the noise creates thousands of short speckle edges; after smoothing, mostly
% the real outlines remain.

%% Exercise 6: segment and measure
C = imread('coins.png');

% Otsu threshold
BW0 = imbinarize(C, graythresh(C));

% Clean the mask
BW1 = imopen(BW0, strel('disk', 3));   % remove small specks and thin bridges
BW2 = imfill(BW1, 'holes');            % fill dark holes inside the coins
BW3 = bwareaopen(BW2, 100);            % remove leftover blobs < 100 pixels

figure;
montage({BW0, BW1, BW2, BW3}, 'Size', [1 4]);
title('Otsu | imopen | imfill | bwareaopen');

% Label and measure
[Lab, nCoins] = bwlabel(BW3);
stats = regionprops(Lab, 'Area', 'Centroid');
areas = [stats.Area]';
fprintf('Number of coins found: %d\n', nCoins);
Coin_table = table((1:nCoins)', areas, 'VariableNames', {'Coin', 'Area_px'})
[maxArea, iMax] = max(areas);
fprintf('Largest coin: #%d with %d pixels\n', iMax, maxArea);

figure;
imshow(labeloverlay(C, Lab));
title('Labelled coins');
hold on;
for k = 1:nCoins
    text(stats(k).Centroid(1), stats(k).Centroid(2), num2str(k), ...
        'Color', 'w', 'FontWeight', 'bold', 'HorizontalAlignment', 'center');
end
hold off;

%% Exercise 6: think about
% *Which step removed what:* Otsu leaves dark holes inside some coins (where the
% coin surface is darker or reflects) and a few small bright specks. imopen
% removes the specks and thin connections, imfill fills the holes inside the
% coins, and bwareaopen removes any small blobs that are left.
%
% *Largest coin:* printed above (number shown on the labelled figure).
%
% *Touching coins:* in coins.png the coins do not touch, so each one gets its own
% label and the count matches the number of coins. If two coins touched, they
% would merge into one connected region and be counted as one large coin; a


<img width="1920" height="1080" alt="Screenshot (171)" src="https://github.com/user-attachments/assets/0638663c-8f9c-4908-9c28-657030eda6ec" />
<img width="1920" height="1080" alt="Screenshot (170)" src="https://github.com/user-attachments/assets/dcdfac6a-754c-450e-ab95-8c6822a39f2e" />
<img width="1920" height="1080" alt="Screenshot (169)" src="https://github.com/user-attachments/assets/49e74e43-b3db-474b-85a9-3bcd0a37e299" />
<img width="1920" height="1080" alt="Screenshot (168)" src="https://github.com/user-attachments/assets/2573b016-c106-4210-9f89-c1bb93687ebe" />
