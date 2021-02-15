Byte[,,] inImage = null, outImage = null;
        int inH, inW, outH, outW;
        String fileName;
        Bitmap paper, bitmap;

        const int RGB = 3, RR = 0, GG = 1, BB = 2;
        //bool mouseYN = false;
        int mouseYN = 0; // 0이면 Off, 나머지는 ON
        int sx, sy, ex, ey;

        int[] mx = new int[500];
        int[] my = new int[500];
        int mCount = 0;

        //메뉴 이벤트 함수부
        private void 열기ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            openImage();
        }

        private void 저장ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            SaveImage();
        }

        private void 동일영상ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            equal_image();
        }

        private void 밝게어둡게ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            bright_image();
        }

        private void 흑백이미지ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            bw_image();
        }

        private void 반전이미지ToolStripMenuItem_Click_1(object sender, EventArgs e)
        {
            rev_Image();
        }

        private void 그레이스케일ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            grayscale();
        }

        private void 포스터라이징ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            posterising();
        }

        private void 상하반전ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            Pd_Image();
        }

        private void 좌우반전ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            level_Image();
        }

        private void 회전ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            rot_Image();
        }

        private void 확대ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            zoom_in();
        }

        private void 축소이미지ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            zoom_out();
        }

        private void 엠보싱ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            emboss_image();
        }

        private void 블러링ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            blurr_image();
        }

        private void 샤프닝ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            sharpen_Image();
        }

        private void 히스토그램스트래칭ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            histo_stretch();
        }

        private void 앤드인탐색ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            end_in_search();
        }

        private void 히스토그램평활화ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            histo_equal();
        }

        private void 히스토그램그리기ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            draw_histogram();
        }

        private void Form1_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Control)
            {
                switch (e.KeyCode)
                {
                    case Keys.O:
                        openImage(); break;
                }
            }
            switch (e.KeyCode)
            {
                case Keys.E:
                    emboss_image(); break;
            }
        }

        //공통 함수부
        void openImage()
        {
            OpenFileDialog ofd = new OpenFileDialog();  // 객체 생성
            ofd.DefaultExt = "";
            ofd.Filter = "칼라 필터 | *.png; *.jpg; *.bmp; *.tif";
            if (ofd.ShowDialog() != DialogResult.OK)
                return;
            fileName = ofd.FileName;
            // 파일 --> 비트맵(bitmap)
            bitmap = new Bitmap(fileName);
            // 중요! 입력이미지의 높이, 폭 알아내기
            inW = bitmap.Height;
            inH = bitmap.Width;
            inImage = new byte[RGB, inH, inW]; // 메모리 할당
            // 비트맵(bitmap) --> 메모리 (로딩)
            for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                {
                    Color c = bitmap.GetPixel(i, k);
                    inImage[RR, i, k] = c.R;
                    inImage[GG, i, k] = c.G;
                    inImage[BB, i, k] = c.B;
                }
            equal_image();
        }

        void SaveImage()
        {
            SaveFileDialog sfd = new SaveFileDialog();
            sfd.DefaultExt = "";
            sfd.Filter = "PNG File(*.png) | *.png";
            if (sfd.ShowDialog() != DialogResult.OK)
                return;
            String saveFname = sfd.FileName;
            Bitmap image = new Bitmap(outH, outW); // 빈 비트맵(종이) 준비
            for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    Color c;
                    int r, g, b;
                    r = outImage[0, i, k];
                    g = outImage[1, i, k];
                    b = outImage[2, i, k];
                    c = Color.FromArgb(r, g, b);
                    image.SetPixel(i, k, c);  // 종이에 콕콕 찍기
                }
            // 상단에 using System.Drawing.Imaging; 추가해야 함
            image.Save(saveFname, ImageFormat.Png); // 종이를 PNG로 저장
            toolStripStatusLabel1.Text = saveFname + "으로 저장됨.";
        }

        void equal_image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // *** 진짜 영상처리 알고리즘을 구현 ***

            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                    outImage[rgb, i, k] = inImage[rgb,i, k];
                    }
            /////////////////////////////////////////////
           displayImage();
        }

        double getValue()
        {
            subForm1 sub = new subForm1(); // 서브폼 준비
            if (sub.ShowDialog() == DialogResult.Cancel)
                return 0;
            double value = (int)sub.numericUpDown1.Value;
            return value;
        }

        void displayImage()
        {
            // 벽, 게시판, 종이 크기 조절
            paper = new Bitmap(outH, outW); // 종이
            pictureBox1.Size = new Size(outH, outW); // 캔버스
            this.Size = new Size(outH + 20, outW + 80); // 벽

            Color pen; // 펜(콕콕 찍을 용도)
            for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    byte r = outImage[RR, i, k]; // 잉크(색상값)
                    byte g = outImage[GG, i, k]; // 잉크(색상값)
                    byte b = outImage[BB, i, k]; // 잉크(색상값)
                    pen = Color.FromArgb(r, g, b); // 펜에 잉크 묻히기
                    paper.SetPixel(i, k, pen); // 종이에 콕 찍기
                }
            pictureBox1.Image = paper; // 게시판에 종이를 붙이기.
            toolStripStatusLabel1.Text =
                outH.ToString() + "x" + outW.ToString() + "  " + fileName;
        }

        //영상처리 함수부

        void bright_image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
            int value = (int)getValue();
            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        if (inImage[rgb, i, k] + value > 255)
                            outImage[rgb, i, k] = 255;
                        else if (inImage[rgb, i, k] + value < 0)
                            outImage[rgb, i, k] = 0;
                        else
                            outImage[rgb, i, k] = (byte)(inImage[rgb, i, k] + value);
                    }
            displayImage();
        }

        private void pictureBox1_MouseDown(object sender, MouseEventArgs e)
        {
            if (mouseYN == 0)
                return;
            else if (mouseYN == 1) // 사각형 범위
            {
                sx = e.X; sy = e.Y;
            }
            else if (mouseYN == 2) // 임의 범위
            {
                if (e.Button == MouseButtons.Left)
                {
                    mx[mCount] = e.X;
                    my[mCount] = e.Y;
                    mCount++;
                }
                else
                { // 오른쪽 버튼 클릭시..

                    if (inImage == null)
                        return;
                    // 중요! 출력이미지의 높이, 폭을 결정 --> 알고리즘에 영향
                    outH = inH; outW = inW;
                    outImage = new byte[RGB, outH, outW];
                    // *** 진짜 영상처리 알고리즘을 구현 ***
                    for (int rgb = 0; rgb < RGB; rgb++)
                        for (int i = 0; i < inH; i++)
                            for (int k = 0; k < inW; k++)
                            {
                                if (pointInPolygon(k, i))
                                    outImage[rgb, i, k] = (byte)(255 - inImage[rgb, i, k]);
                                else
                                    outImage[rgb, i, k] = inImage[rgb, i, k];
                            }
                    /////////////////////////////////////////////
                    displayImage();

                    // 초기화
                    mouseYN = 0;
                    mCount = 0;
                }
            }
        }
                private bool pointInPolygon(int pntX, int pntY)
                {
                    bool inPoly = false;
                    int iCrosses = 0; // 교차 횟수 
                    for (int i = 0; i < mCount; i++)
                    {
                        int j = (i + 1) % mCount;
                        if ((my[i] > pntY) != (my[j] > pntY))
                        {
                            double atX = ((((double)mx[j] - mx[i]) / ((double)my[j] - my[i])) * ((double)pntY - my[i])) + (double)mx[i];
                            if (pntX < atX)
                                iCrosses++;
                        }
                    } // 홀수면 내부, 짝수면 외부에 있음 
                    if (0 == (iCrosses % 2))
                        inPoly = false;
                    else inPoly = true;
                    return inPoly;

                }

        private void pictureBox1_MouseUp(object sender, MouseEventArgs e)
        {
            if (mouseYN == 0)
                return;
            else if (mouseYN == 1)
            {
                ex = e.X; ey = e.Y;

                if (sx > ex)
                {
                    int tmp = sx; sx = ex; ex = tmp;
                }
                if (sy > ey)
                {
                    int tmp = sy; sy = ey; ey = tmp;
                }
           // 반전하기.
             switch (mouseYN)
                {
                    case 1: rev_Image(); break;
                }
                mouseYN = 0;
            }

         //if (inImage == null)
         // return;
         //// 중요! 출력이미지의 높이, 폭을 결정 --> 알고리즘에 영향
         //outH = inH; outW = inW;
         //outImage = new byte[outH, outW];
         //// *** 진짜 영상처리 알고리즘을 구현 ***
         //for (int i = 0; i < inH; i++)
         // for (int k = 0; k < inW; k++)
         // {
         // if ((sx <= k && k <= ex) && (sy <= i && i <= ey) )
         // outImage[i, k] = (byte)(255 - inImage[i, k]);
         // else
         // outImage[i, k] = inImage[i, k];
         // }
         ///////////////////////////////////////////////
         //displayImage();​
        }

        void bw_image()
        {
            if (inImage == null)
                return;
            int sum = 0;
            for (int rgb = 0; rgb < RGB; rgb++)
              for (int i = 0; i < inH; i++)
            {
                for (int k = 0; k < inW; k++)
                {
                    sum += outImage[rgb, i, k];
                }
            }
            double avg = (double)sum / (inH * inW);   // 평균
            for (int rgb = 0; rgb < RGB; rgb++)
              for (int i = 0; i < inH; i++)
                {
                for (int k = 0; k < inW; k++)
                {
                    if (outImage[rgb, i, k] < avg)
                        outImage[rgb, i, k] = 0;
                    else
                        outImage[rgb, i, k] = 255;

                }
            }
            displayImage();
        }

        void rev_Image() //반전
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
            if (mouseYN == 0)
            {
                sx = 0; ex = inH;
                sy = 0; ey = inW;
            }
            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                {
                    if ((sx <= k && k <= ex) && (sy <= i && i <= ey))
                        outImage[rgb, i, k] = (byte)(255 - inImage[rgb, i, k]);
                    else
                        outImage[rgb, i, k] = inImage[rgb, i, k];
                }
            /////////////////////////////////////////////
            displayImage();
        }

        private void 반전마우스ToolStripMenuItem_Click_1(object sender, EventArgs e)
        {
            mouseYN = 1;
        }

        private void 반전임의범위ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            mouseYN = 2;
        }

        private void 채도변경ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            change_satur();
        }

        void grayscale()
        {
           if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
                    // *** 진짜 영상처리 알고리즘을 구현 ***
             for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)

                {
                    int hap = inImage[RR, i, k] + inImage[GG, i, k] + inImage[BB, i, k];
                    byte rgb = (byte) (hap / 3.0);
                    outImage[RR, i, k] = rgb;   
                    outImage[GG, i, k] = rgb;
                    outImage[BB, i, k] = rgb;
                }
            /////////////////////////////////////////////
            displayImage();
        }

        void posterising()
        {
            if (inImage == null)
                return;

            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            byte cutLevel = 40;       //경계값
            int cut = 255 / cutLevel;   //나눈수


            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    //칼라 -> 흑백 -> 칼라
                    int hap = inImage[RR, i, k] + inImage[GG, i, k] + inImage[BB, i, k];
                    int avg = hap / RGB;                    //흑백 값

                    //구간 나누기
                    for (int m = 0; m <= cut; m++)
                    {
                        if (cutLevel * m <= avg && avg < cutLevel * (m + 1))
                        {
                            if (avg < cutLevel)
                            {
                                avg = 0;
                                break;
                            }

                            else
                            {
                                avg = (byte)(cutLevel * m);
                                break;
                            }
                        }
                    }

                    //RGB에 비율 정하기
                    double RRate = (double)inImage[RR, i, k] / hap;   //칼라에서 R비율
                    double GRate = (double)inImage[GG, i, k] / hap;   //칼라에서 G비율
                    double BRate = (double)inImage[BB, i, k] / hap;   //칼라에서 B비율

                    outImage[RR, i, k] = (byte)(avg * RGB * RRate);
                    outImage[GG, i, k] = (byte)(avg * RGB * GRate);
                    outImage[BB, i, k] = (byte)(avg * RGB * BRate);

                }

            /////////////////////////////////////////////
            displayImage();
        }

        void Pd_Image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // *** 진짜 영상처리 알고리즘을 구현 ***

            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        outImage[rgb, i, k] = inImage[rgb, i, (outW - 1 - k)];
                    }
            /////////////////////////////////////////////
            displayImage();
        }

        void level_Image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // *** 진짜 영상처리 알고리즘을 구현 ***

            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        outImage[rgb, i, k] = inImage[rgb, (inW - 1 - i), k];
                    }
            /////////////////////////////////////////////
            displayImage();
        }
        void rot_Image()
        {      // 회전

            if (inImage == null)
                return;
            double val;
            int value = (int)getValue();
            double pi = 3.141592;
            double seta = pi / (180.0 / value);
            int center_x = inW / 2;
            int center_y = inH / 2;
            int x;
            int y;
            for (int rgb = 0; rgb < RGB; rgb++)
               for (int i = 0; i < inH; i++)
            {
                for (int k = 0; k < inW; k++)
                {
                    x = (int)((i - center_y) * Math.Sin(seta) + (k - center_x) * Math.Cos(seta) + center_x);
                    y = (int)((i - center_y) * Math.Cos(seta) - (k - center_x) * Math.Sin(seta) + center_y);
                    if (y < 0 || y >= inH)
                    {
                        // 회전된좌표가출력영상을위한배열값을넘어갈때
                        val = 0;
                    }
                    else if (x < 0 || x >= inW)
                    {
                        // 회전된좌표가출력영상을위한배열값을넘어갈때
                        val = 0;
                    }
                    else
                    {
                        val = inImage[rgb, y, x];
                    }
                    outImage[rgb, i, k] = (byte)(val);
                }
            }
            displayImage();
        }

        void zoom_in()  //확대 알고리즘
        {
            Console.WriteLine("확대 배율-->");
            int scale = (int)getValue();
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH * scale;
            outW = inW * scale;
            outImage = new byte[RGB, outH, outW];
            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                 for (int i = 0; i < outH; i++)
                      for (int k = 0; k < outW; k++)
                {
                    outImage[rgb, i, k] = inImage[rgb, (i / scale), (k / scale)];
                }
            displayImage();
        }

        void zoom_out() // 축소 알고리즘
        {
            Console.WriteLine("축소 배율-->");
            int scale = (int)getValue();
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향

            outH = inH / scale;
            outW = inW / scale;
            outImage = new byte[RGB, outH, outW];
            int sum;
            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
             {
                for (int k = 0; k < inW; k++)
                {
                    outImage[rgb, (i / scale), (k / scale)] = inImage[rgb, i, k];
                }
            }
            displayImage();
        }

        void emboss_image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
            // 화소영역 처리
            // 중요! 마스크 결정
            const int MSIZE = 3;
            double[,] mask = {        { -1.0, 0.0, 0.0 },
                                            {  0.0, 0.0, 0.0 },
                                            {  0.0, 0.0, 1.0 }   };
            // 임시 입력, 출력 메모리 할당
            double[,] tmpInput = new double[inH + 2, inW + 2];
            double[,] tmpOutput = new double[outH, outW];

            // 임시 입력을 중간값인 127로 초기화
            for (int i = 0; i < inH + 2; i++)
                for (int k = 0; k < inW + 2; k++)
                    tmpInput[i, k] = 127.0;
            // 입력 --> 임시 입력
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                    tmpInput[i + 1, k + 1] = inImage[rgb, i, k];

            // *** 진짜 영상처리 알고리즘을 구현 ***
            double S = 0.0;
            for (int i = 0; i < inH; i++)
            {
                for (int k = 0; k < inW; k++)
                {
                    for (int m = 0; m < MSIZE; m++)
                        for (int n = 0; n < MSIZE; n++)
                            S += tmpInput[i + m, k + m] * mask[m, n];
                    tmpOutput[i, k] = S;
                    S = 0.0;
                }
            }
            // 후처리 : Mask의 합이 0이면 127을 더하기.
            for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                    tmpOutput[i, k] += 127.0;
            // 임시 출력 --> 원래 출력
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    double d = tmpOutput[i, k];
                    if (d > 255.0)
                        d = 255.0;
                    else if (d < 0.0)
                        d = 0.0;
                    outImage[rgb, i, k] = (byte)d;
                }
            /////////////////////////////////////////////
            displayImage();
        }

        void blurr_image()
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
            // 화소영역 처리
            // 중요! 마스크 결정
            const int MSIZE = 3;
            double[,] mask = {        {  1/9.0, 1/9.0, 1/9.0 },
                                            {  1/9.0, 1/9.0, 1/9.0  },
                                            {  1/9.0, 1/9.0, 1/9.0  }   };
            // 임시 입력, 출력 메모리 할당
            double[,] tmpInput = new double[inH + 2, inW + 2];
            double[,] tmpOutput = new double[outH, outW];

            // 임시 입력을 중간값인 127로 초기화
            for (int i = 0; i < inH + 2; i++)
                for (int k = 0; k < inW + 2; k++)
                    tmpInput[i, k] = 127.0;
            // 입력 --> 임시 입력
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                    tmpInput[i + 1, k + 1] = inImage[rgb, i, k];
            // *** 진짜 영상처리 알고리즘을 구현 ***
            double S = 0.0;
            for (int i = 0; i < inH; i++)
            {
                for (int k = 0; k < inW; k++)
                {
                    for (int m = 0; m < MSIZE; m++)
                        for (int n = 0; n < MSIZE; n++)
                            S += tmpInput[i + m, k + n] * mask[m, n];
                    tmpOutput[i, k] = S;
                    S = 0.0;
                }
            }
            // 후처리 : Mask의 합이 0이면 127을 더하기.
            //for (int i = 0; i < outH; i++)
            //    for (int k = 0; k < outW; k++)
            //        tmpOutput[i, k] += 127.0;
            // 임시 출력 --> 원래 출력
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    double d = tmpOutput[i, k];
                    if (d > 255.0)
                        d = 255.0;
                    else if (d < 0.0)
                        d = 0.0;
                    outImage[rgb, i, k] = (byte)d;
                }
            /////////////////////////////////////////////
            displayImage();
        }

        void sharpen_Image()
        {
            if (inImage == null)
                return;
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];
            const int MSIZE = 3;
            double[,] mask = { { 0.0, -1.0, 0.0 },
                               { -1.0, 5.0, -1.0 },
                               { 0.0, -1.0, 0.0 } };

            double[,] tmpInput = new double[inH + 2, inW + 2];
            double[,] tmpOutput = new double[outH, outW];

                for (int i = 0; i < inH + 2; i++)
                for (int k = 0; k < inW + 2; k++)
                    tmpInput[i, k] = 127.0;

            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                    tmpInput[i + 1, k + 1] = inImage[rgb, i, k];
            double S = 0.0;
            for (int i = 0; i < inH; i++)
            {
                for (int k = 0; k < inW; k++)
                {
                    for (int m = 0; m < MSIZE; m++)
                        for (int n = 0; n < MSIZE; n++)
                            S += tmpInput[i + m, k + n] * mask[m, n];
                    tmpOutput[i, k] = S;
                    S = 0.0;

                }
            }
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    double d = tmpOutput[i, k];
                    if (d > 255.0)
                        d = 255.0;
                    else if (d < 0.0)
                        d = 0.0;
                    outImage[rgb, i, k] = (byte)d;
                }
            /////////////////////////////////////////////
            displayImage();
        }

        void draw_histogram()
        {
            long[] rhisto = new long[256];
            long[] ghisto = new long[256];
            long[] bhisto = new long[256];

            for (int i = 0; i < outH; i++)
                for (int k = 0; k < outW; k++)
                {
                    rhisto[outImage[RR, i, k]]++;
                    ghisto[outImage[GG, i, k]]++;
                    bhisto[outImage[BB, i, k]]++;
                }

            histoform hform = new histoform(rhisto, ghisto, bhisto);
            hform.ShowDialog();
        }
        void histo_stretch() // 히스토그램 스트래칭
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // 수식 :
            // Out = (In - min) / (max - min) * 255.0
            byte min_val = inImage[0, 0, 0], max_val = inImage[0, 0, 0];
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                {
                    if (min_val > inImage[rgb, i, k])
                        min_val = inImage[rgb, i, k];
                    if (max_val < inImage[rgb, i, k])
                        max_val = inImage[rgb, i, k];
                }
            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                for (int k = 0; k < inW; k++)
                {
                    // Out = (In - min) / (max - min) * 255.0
                    outImage[rgb, i, k] = (byte)
                        ((double)(inImage[rgb, i, k] - min_val) / (max_val - min_val) * 255.0);
                }
            /////////////////////////////////////////////
            displayImage();
        }

        void end_in_search() // 앤드인 탐색
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // 수식 :
            // Out = (In - min) / (max - min) * 255.0
            byte min_val = inImage[0, 0, 0], max_val = inImage[0, 0, 0];
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        if (min_val > inImage[rgb, i, k])
                            min_val = inImage[rgb, i, k];
                        if (max_val < inImage[rgb, i, k])
                            max_val = inImage[rgb, i, k];
                    }
            // min, max 값을 강제 변경
            min_val += 50;
            max_val -= 50;

            // *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        // Out = (In - min) / (max - min) * 255.0
                        double value =
                            ((double)(inImage[rgb, i, k] - min_val) / (max_val - min_val) * 255.0);
                        if (value > 255)
                            value = 255;
                        else if
                            (value < 0)
                            value = 0;
                        outImage[rgb, i, k] = (byte)value;
                    }

            displayImage();
        }

        void histo_equal() // 히스토그램 평활화
        {
            if (inImage == null)
                return;
            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            //1.히스토그램 생성
            int[] histo = new int[256];
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                        histo[inImage[rgb, i, k]]++;

            //2.누적 히스토그램
            int[] sumHisto = new int[256];
            int sum_val = 0;
            for (int i = 0; i < 256; i++)
            {
                sum_val += histo[i];
                sumHisto[i] = sum_val;
            }

            //3.정규화된 누적 히스토그램
            //n=(누적합/(행*열)) * 255.0
            double[] normalHisto = new double[256];
            for (int i = 0; i < 256; i++)
            {
                normalHisto[i] = ((Double)sumHisto[i] / (RGB * inH * inW)) * 255.0;
            }

            /// *** 진짜 영상처리 알고리즘을 구현 ***
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        outImage[rgb, i, k] = (byte)normalHisto[inImage[rgb, i, k]];
                    }
            /////////////////////////////////////////////
            displayImage();
        }

        void change_satur()
        {
            if (inImage == null)
                return;

            // 중요! 출력이미지의 높이, 폭을 결정  --> 알고리즘에 영향
            outH = inH; outW = inW;
            outImage = new byte[RGB, outH, outW];

            // *** 진짜 영상처리 알고리즘을 구현 ***
            Color c; // 한점 색상 모델
            double hh, ss, vv; // 색상, 채도, 밝기
            int rr, gg, bb; // 레드 그린 블루
            for (int rgb = 0; rgb < RGB; rgb++)
                for (int i = 0; i < inH; i++)
                    for (int k = 0; k < inW; k++)
                    {
                        rr = inImage[RR, i, k];
                        gg = inImage[GG, i, k];
                        bb = inImage[BB, i, k];
                        // RGB ---> HSV(HSB)
                        c = Color.FromArgb(rr, gg, bb);
                        hh = c.GetHue();
                        ss = c.GetSaturation();
                        vv = c.GetBrightness();

                        // 채도 올리기
                        ss += 150.0;

                        //HSV --> RGB
                        HsvToRgb(hh, ss, vv, out rr, out gg, out bb);

                        outImage[rgb, i, k] = (byte)rr;
                        outImage[rgb, i, k] = (byte)gg;
                        outImage[rgb, i, k] = (byte)bb;
                    }
            /////////////////////////////////////////////
            displayImage();
        }

        void HsvToRgb(double h, double S, double V, out int r, out int g, out int b)
        {
            double H = h;
            while (H < 0) { H += 360; };
            while (H >= 360) { H -= 360; };
            double R, G, B;
            if (V <= 0)
            { R = G = B = 0; }
            else if (S <= 0)
            {
                R = G = B = V;
            }
            else
            {
                double hf = H / 60.0;
                int i = (int)Math.Floor(hf);
                double f = hf - i;
                double pv = V * (1 - S);
                double qv = V * (1 - S * f);
                double tv = V * (1 - S * (1 - f));
                switch (i)
                {
                    case 0:
                        R = V;
                        G = tv;
                        B = pv;
                        break;
                    case 1:
                        R = qv;
                        G = V;
                        B = pv;
                        break;
                    case 2:
                        R = pv;
                        G = V;
                        B = tv;
                        break;
                    case 3:
                        R = pv;
                        G = qv;
                        B = V;
                        break;
                    case 4:
                        R = tv;
                        G = pv;
                        B = V;
                        break;
                    case 5:
                        R = V;
                        G = pv;
                        B = qv;
                        break;
                    case 6:
                        R = V;
                        G = tv;
                        B = pv;
                        break;
                    case -1:
                        R = V;
                        G = pv;
                        B = qv;
                        break;
                    default:
                        R = G = B = V;
                        break;
                }
            }
            r = CheckRange((int)(R * 255.0));
            g = CheckRange((int)(G * 255.0));
            b = CheckRange((int)(B * 255.0));

            int CheckRange(int i)
            {
                if (i < 0) return 0;
                if (i > 255) return 255;
                return i;
            }
        }
