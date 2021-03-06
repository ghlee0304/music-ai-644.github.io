# #4 Parallel WaveGAN: A fast waveform generation model based on generative adversarial networks with multi-resolution spectrogram

- Code: https://github.com/kan-bayashi/ParallelWaveGAN
- Paper: https://arxiv.org/pdf/1910.11480.pdf
- Sample: https://r9y9.github.io/demos/projects/icassp2020/
- Tags: Vocoder

# Abstract

기존의 parallel wavenet 과는 다르게 teacher network에서 distillation을 할 필요가 없고 빠르게 음성을 만들어낼 수 있는 보코더를 제안하였다. 논문에서 제안하는 non-autoregressive WaveNet은 multi-resolution spectrogram과 adversarial loss function을 결합하여 모델을 학습하였다. GAN을 이용함으로써 좀 더 작은 모델로 빠르고 고품질의 음성을 생성할 수 있다는 장점을 가지고 있다.  

- 특이사항
    - ICASSP 2020 제출
- **Introduction**
    
    WaveNet, IAF 모델 간단히 설명이 있다. 
    
    본 논문의 contribution은 다음과 같다. 
    
    - multi-resolution STFT loss와 waveform-domain adversarial loss를 결합하여 학습하는 방법을 이용하여 효율적으로 학습이 가능하였다.
    - Parallel WaveGAN은 teacher-student framework 없이 간단하게 학습 되며 training과 inference의 시간을 줄였다.
    - Parallel WaveGAN과 결합한 transformer 기반의 TTS 모델에 대하여 ClariNet model 보다 높은 4.16의 MOS를 기록하였다.
- **Method**
    - **Parallel waveform generation based on GAN**
        1. WaveNet 기반의 generator는 noise input과 보조(auxilary) feature인 mel-spectrogram을 입력으로 받아서 waveform을 parallel하게 만든다.  
        2. generator가 기존의 WaveNet과 다른 점은 다음과 같다. 
            - causal convolution 대신에 non-causal convolution을 사용하였다.
            - input을 Gaussian distribution으로부터 랜덤한 noise를 사용하였다.
            - training과 inference step 모두 non-autoregressive하게 작동한다.
        3. adversarial loss는 다음과 같고, z는 white noise이다. 
            
            $$
            L_{adv}(G, D)=\mathbb{E}_{z\sim N(0,1)}\left [ \left ( 1-D(G(z))^2\right )\right ] 
            $$
            
        4. discriminator는 sample을 fake인지 real인지 판단을 하며 optimization criterion 은 다음과 같다. 
            
            $$
            L_D\left ( G, D \right )=\mathbb{E}_{x\sim p_{data}}\left [ (1-D(x))^2)\right ] + \mathbb{E}_{z\sim N(0,1)}\left [ D(G(z))^2 \right ]
            $$
            
    - **Multi-resolution STFT auxiliary loss**
        1. STFT loss는 저자의 이전 논문에 사용되었던 loss인 spectral convergence loss와 log STFT magnitude loss를 결합한 loss가 사용되었다. 
            
            ![Untitled.png]("/_posts/Parallel WaveGAN A fast waveform generation model based on generative adversarial networks with multi-resolution spectrogram/Untitled.png")
            
        2. spectral convergence loss 
            
            ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%201.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%201.png)
            
        3. log STFT magnitude loss 
            
            ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%202.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%202.png)
            
        4. 논문의 저자들은 다중 resolution STFT auxiliary loss 를 사용하였다. 해당 방법은 generator가 time-frequency 특징을 더 잘 학습하도록 해주었으며 오버 피팅을 막아주었다. (해당 방법은 uni wavenet에서 아이디어를 얻지 않았을까 추측해본다.)
            
            ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%203.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%203.png)
            
        5. 최종 loss 
            
            ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%204.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%204.png)
            
            ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%205.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%205.png)
            
- **Experiment**
    - **Datasets**
        1. female professional Japanese speaker
        2. sampling rate : 24kHz (16 bits)
        3. training : 11,449 utterances (23.09 hours) | test : 250 utterances (0.35 hours) | validation : 250 utterance (0.34 hours)
        4. 80 band log-mel spectrograms with band-limited frequency range(70 to 8000 Hz)
        5. frame lengths : 50 (ms), shift lengths :12.5 (ms) 
    - **Model settings**
        
        **generator** 
        
        - 30 layers of dilated residual convolution blocks with exponentially increasing three dilation cycles
        - residual and skip channels : 각각 64
        - convolution filter size : 3
        
        **discriminator** 
        
        - 10 layers of non-causal dilated 1D convolutions with leaky ReLU activation function ($\alpha=0.2$)
        - stride는 첫번째와 마지막 layer를 제외하고 1부터 선형적으로 8까지 증가 시켜서 사용
        - channel과 filter size는 generator와 동일하게 사용
        - generator와 discriminator 모두 weight normalization 적용
    - **Training and Inference**
        - $\lambda_{adv}=4.0$
        - total training step : 400K
        - RAdam optimizer ($\epsilon=1e^{-6}$)
        - discriminator는 처음 100K는 고정하고 이후 jointly 하게 학습
        - mini batch size : 8
        - the length of audio clip : 24K time samples (1.0 second)
        - initial learning rate : 0.0001 (generator), 0.00005 (discriminator)
        - learning rate decay : (half) 200K steps
        - ClariNet / WaveNet parameter는 논문 참조
- **Results**
    - **System Results**
        
        ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%206.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%206.png)
        
    - **MOS w/ TTS**
        
        ![#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%207.png](#4%20Parallel%20WaveGAN%20A%20fast%20waveform%20generation%20mod%20b26e708d1b5d4340a39696700b9174ad/Untitled%207.png)
        
- **Conclusion**